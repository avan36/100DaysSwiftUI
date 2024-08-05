# Day 29

### Introduction to Lists

```swift
List {
            Text("Hello, world!")
            Text("Hello, world!")
            Text("Hello, world!")
 }
```

### Dynamic Content and Sections

```swift
List {
 Section("Section 1") {
     Text("Static Row 1")
        ForEach(0..<5) {
                Text("Dynamic Row \($0)")
           }
        }
	 }
 }
```

### Loop over array in ListView

```swift
struct ContentView: View {
    let people = ["Finn", "Leia", "Luke", "Rey"]
    var body: some View {
        List {
            Text("Static Row")
            ForEach(people, id: \.self) {
                Text($0)
            }
        }
        
    }
}
```

### Loading resources from app bundle

In *same struct as main one*, we can write this code:

```swift
func testBundles() {
        if let fileURL = Bundle.main.url(forResource: "somefile", withExtension: "txt") {
            if let fileContents = try? String(contentsOf: fileURL) {
                //We found file in bundle and loaded into a string
                
            }
            //if succeeds, we found the file in bundle
        }
    }
```

### Get Random element from array

```swift
let letter = letters.randomElement()
```

### Turn String into array

```swift
let letters = input.components(separatedBy: " ") 
```

### Turning into string

```swift

import SwiftUI

struct ContentView: View {
    let people = ["Finn", "Leia", "Luke", "Rey"]
    var body: some View {
        List {
            Text("Static Row")
            ForEach(people, id: \.self) {
                Text($0)
            }
        }
        
    }
    func testStrings() {
        let input = """
        a
        b
        c
        """
        let letters = input.components(separatedBy: "\n") // Will give us 3 item array separated by spaces
        let letter = letters.randomElement()
        let trimmed = letter?.trimmingCharacters(in: .whitespacesAndNewlines)
    }
            
}
```

### Checking for mispelling in User entered string

```swift
func testStrings() {
        let word = "swift"
        let checker = UITextChecker()
        
        let range = NSRange(location: 0, length: word.utf16.count)
        //utf16 is a character encoding, used here so Objective C can understand how swift strings are stored
        let mispelledRange = checker.rangeOfMisspelledWord(in: word, range: range, startingAt: 0, wrap: false, language: "en")
        //This sends back objective c string range telling us where the mispelled word was found.
        
        let allGood = mispelledRange.location == NSNotFound
        //Returned NSNotFound if not spelling mistake found
    }
```

### Final

```python
//
//  ContentView.swift
//  WordScramble
//
//  Created by Ambrose V on 17/06/2024.
//

import SwiftUI

struct ContentView: View {
    let people = ["Finn", "Leia", "Luke", "Rey"]
    var body: some View {
        List {
            Text("Static Row")
            ForEach(people, id: \.self) {
                Text($0)
            }
        }
        
    }
    
    func testStrings() {
        let word = "swift"
        let checker = UITextChecker()
        
        let range = NSRange(location: 0, length: word.utf16.count)
        //utf16 is a character encoding, used here so Objective C can understand how swift strings are stored
        let mispelledRange = checker.rangeOfMisspelledWord(in: word, range: range, startingAt: 0, wrap: false, language: "en")
        //This sends back objective c string range telling us where the mispelled word was found.
        
        let allGood = mispelledRange.location == NSNotFound
        //Returned NSNotFound if not spelling mistake found
        
    }
            
}

#Preview {
    ContentView()
}

```