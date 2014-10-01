Various Go utilities related to MapleStory (encryption, packets, and so on).

Getting started
============
Make sure that you have git and go installed and run

	go get github.com/jteeuwen/go-pkg-xmlx
	go get github.com/Francesco149/maplelib

You can also manually clone the repository anywhere you want by running

	git clone https://github.com/Francesco149/maplelib.git
    
To verify that jteeuwen's xml library and my library are installed and working, run

	go test github.com/jteeuwen/go-pkg-xmlx/...
	go test github.com/Francesco149/maplelib/...
    
Examples
============

Building, encrypting, decrypting and decoding a packet:

	package main

	import (
		"fmt"
		"crypto/rand"
	)
	import "github.com/Francesco149/maplelib"

	func main() {
		// initialize random initialization vector
		initializationRandomness := [4]byte{}
		rand.Read(initializationRandomness[:])
	
		// initialize crypto for maple v62
		crypt := maplelib.NewCrypt(initializationRandomness, 62)
		fmt.Println("crypt =", crypt)

		// build a new packet
		p := maplelib.NewPacket()
		p.Encode4(0x00000000) // placeholder for encrypted header
		p.Encode2(255)
		p.EncodeString("Hello world!")
		p.Encode4s(-5000)
		fmt.Println("p =", p)
	
		// encrypt the packet
		crypt.Encrypt([]byte(p))
		fmt.Println("encrypted p =", p)
	
		// get the original packet length from the encrypted packet
		// note: you would normally copy packetlen bytes from whatever source 
		// your packets are coming from to a new packet
		packetlen := maplelib.GetPacketLength(p)
		fmt.Println("decrypted length:", packetlen)
	
		// decrypt the packet
		// (note: you normally have to call .Shuffle after every encrypt/decrypt)
		p = p[4:] // skip first 4 bytes (encrypted header)
		crypt.Decrypt([]byte(p))
		fmt.Println("decrypted p =", p)
	
		// decode the stuff we encoded earlier
		it := p.Begin()
	
		word, err := it.Decode2()
		checkError(err)
	
		str, err := it.DecodeString()
		checkError(err)
	
		signed_dword, err := it.Decode4s()
		checkError(err)
	
		fmt.Println("word =", word, "\nstr =", str, "\nsigned_dword =", signed_dword)
	}

	func checkError(err error) {
		if err != nil {
			panic(err)
		}
	}
	

Reading wz xml files:

	package main

	import (
		"fmt"
		"path/filepath"
		"os"
	)

	import "github.com/Francesco149/maplelib/wz"

	func main() {
		// path is just the full or relative path to your wz xml directory
		// I'm just retrieving my test xml files' directory for this example
		path := filepath.Join(os.Getenv("GOPATH"), "src", "github.com",
			"Francesco149", "maplelib", "wz", "testfiles")

		// opening the xml wz root folder
		x, err := wz.NewMapleDataProvider(path)
		checkError(err)

		// opening an img xml file (0003.img.xml)
		img, err := x.Get("TamingMob.wz/0003.img")
		checkError(err)

		// retrieving the INT info/speed value inside 0003.img.xml
		// NOTE: you should normally error check for nil on the returned pointer (val)
		val := wz.GetInt(img.ChildByPath("info/speed")) // returns a *int32
		fmt.Println("TamingMob.wz/0003.img.xml -> info -> speed =", *val)

		// retrieving the FLOAT info/swim value inside 0003.img.xml
		val2 := wz.GetFloat(img.ChildByPath("info/swim")) // return a *float32
		fmt.Println("TamingMob.wz/0003.img.xml -> info -> swim =", *val2)
	}

	func checkError(err error) {
		if err != nil {
			panic(err)
		}
	}
	
    
Documentation
============
You can get the documentation with the built-in godoc 

	godoc github.com/Francesco149/maplelib
    
If you're looking for a specific function or type just use

	godoc github.com/Francesco149/maplelib MyFunction
