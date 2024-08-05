# SwiftUI Learning

# Table of Contents

# Quick How To's

## Variables + Concepts

### Why we use Structs

Structs don't mutate, unlike classes, meaning they have simpler code and faster performacne.

### Modifier order

We need do. **Way to think: **SwiftUi renders view after every single modifier. This means expanding frame won't redraw the background color afterwards.

```
.frame(width: 200, height: 200))
.background(.red)
```

### Print type of view

```
print(type(of: self.body)
```

### Why SwiftUI uses "some" view

SwiftUi needs to know what's inside the view. If we just have text isnide we could say

```
var body: some Text{
Text("HELLO")
}
```

For VStack, Swift creates tuple view that contains two views. View builder wraps multiple views inside a tuple view ocntainer. If we go to definition View protocol, it shows body property marked @

### Create automatically set variable  that updates within a .swift file

```swift
var hasValidAddress: Bool {
        if name.isEmpty || streetAddress.isEmpty || city.isEmpty || zip.isEmpty {
            return false
        }
        return true
}
```

## Files

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

### Loading words from text files

```swift
func startGame() {
        if let startWordsURL = Bundle.main.url(forResource: "start", withExtension: "txt") { //Checking we can find this
            if let startWords = try? String(contentsOf: startWordsURL) {
                let allWords = startWords.components(separatedBy: "\n")
                rootWord = allWords.randomElement() ?? "silkworm"
                return } }
        //If got here, fatal error (should have passed)
        fatalError("Could not load start.txt from bundle")
    }
```

## Structs

### Using structs to carry data

Every time we text something in the textfield, behind the scenes,  an entirely new struct is being created. He says it’s actually very fast. Changing to “class” instead of “struct” doesn’t work here because “private var user” is not changing, the values inside the Class user are changing but not the var user itself. This means the struct cannot spot the change and doesn’t reload the view.

```swift
struct User {
    var firstName = "Bilbo"
    var lastName = "Baggins"
}

struct ContentView: View {
    @State private var user = User()
    var body: some View {
        VStack {
            Text("Your name is \(user.firstName) \(user.lastName)")
            
            TextField("First name", text: $user.firstName)
            TextField("First name", text: $user.firstName)
        }
        .padding()
    }
}
```

### Using classes instead of stricts and Observable

As said above, changing the struct to a class doesn’t work. However, adding @Observable on the line above does work. @Observable says to update any view that relies on the class user when properties change.  By adding import Observation, we can double click on @Observable and it will show all the code imported by SwiftUI. The @State private var just keeps the variable alive and attached to the view. All the watching of the view is handled by Observable, not by @State.

```swift
import Observation
import SwiftUI

@Observable
class User {
    var firstName = "Bilbo"
    var lastName = "Baggins"
}

struct ContentView: View {
    @State private var user = User()
    var body: some View {
        VStack {
            Text("Your name is \(user.firstName) \(user.lastName)")
            
            TextField("First name", text: $user.firstName)
            TextField("First name", text: $user.firstName)
        }
        .padding()
    }
}
```

### UserDefaults

Maximum 512 kb here (a novel worth of words). UserDefaults booleans have a default value of false, so it’s not clear if it’s a value you set or if there’s no value at all. It also takes a little bit of time for iPhone to write to user defaults (could be a few seconds).

### Create UserDefaults Value

```swift
UserDefaults.standard.set(tapCount, forKey: "Tap")
```

### Retrieve UserDefaults Value

```swift
UserDefaults.standard.integer(forKey: "Tap") //Where integer is the type being stored
```

```swift
struct ContentView: View {
    @State private var tapCount = UserDefaults.standard.integer(forKey: "Tap")
    var body: some View {
        Button("Tap Count: \(tapCount)") {
            tapCount += 1
            UserDefaults.standard.set(tapCount, forKey: "Tap")
        }
    }
}
```

### @AppStorage as an alternative way to use UserDefaults

```swift
struct ContentView: View {
    @AppStorage("tapCount") private var tapCount = 0
    var body: some View {
        Button("Tap Count: \(tapCount)") {
            tapCount += 1
        }
    }
}
```

### Codable Structs (Archiving Swift Objects with Codable)

```swift
struct User: Codable {
    let firstName: String
    let lastName: String
}
struct ContentView: View {
    @State private var user = User(firstName: "Taylor", lastName: "Swift")

    var body: some View {
        Button("Save User") {
            let encoder = JSONEncoder()
            
            if let data = try? encoder.encode(user) {
                UserDefaults.standard.set(data, forKey: "UserData")
            }
        }
    }
}
```

### Creating codable structs

```python
struct User: Codable {
    let name: String
    let address: Address //Uses the struct address located below
}

struct Address: Codable {
    let street: String
    let city: String
}

struct ContentView: View {
    var body: some View {
        Button("Decode JSON") {
            let input = """
                {
                    "name": "Taylor Swift",
                    "adress": {
                        "street": "555, Taylor Swift Avenue",
                        "city": "Nashville"
                    }
                }
            """
            
            let data = Data(input.utf8)
            let decoder = JSONDecoder()
            
            if let user = try? decoder.decode(User.self, from: data) {
                print(user.address.street)
            }
        }
    }
}
```

### Converting from JSON Data

```swift
//First option
do {
    let decoder = JSONDecoder()
    decoder.keyDecodingStrategy = .convertFromSnakeCase

    let user = try decoder.decode(User.self, from: data)
    print("Hi, I'm \(user.firstName) \(user.lastName)")
} catch {
    print("Whoops: \(error.localizedDescription)")
} 

//Second option
struct User: Codable {
    enum CodingKeys: String, CodingKey {
        case firstName = "first"
        case lastName = "last"
    }

    var firstName: String
    var lastName: String
}

//Completely custom
struct User: Codable {
    enum CodingKeys: String, CodingKey {
        case firstName = "first"
        case lastName = "last"
        case age
    }

    var firstName: String
    var lastName: String
    var age: Int

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.firstName = try container.decode(String.self, forKey: .firstName)
        self.lastName = try container.decode(String.self, forKey: .lastName)
        self.age = try container.decode(Int.self, forKey: .age)
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(self.firstName, forKey: .firstName)
        try container.encode(self.lastName, forKey: .lastName)
        try container.encode(self.age, forKey: .age)
    }
}
```

### Structs to create Star rating (with .buttonStyle(.plain))

```swift
//
//  RatingView.swift
//  Bookworm
//
//  Created by Ambrose V on 15/07/2024.
//

import SwiftUI

struct RatingView: View {
    @Binding var rating: Int
    
    var label: String = ""
    var maximumRating = 5
    
    var offImage: Image?
    var onImage = Image(systemName: "star.fill")
    
    var offColor = Color.gray
    var onColor = Color.yellow
    var body: some View {
        HStack {
            if label.isEmpty == false {
                Text(label)
            }
            
            ForEach(1..<maximumRating + 1, id: \.self) { number in
                Button {
                    rating = number
                }
            label: {
                image(for: number)
                    .foregroundStyle(number > rating ? offColor : onColor)
            }
            }
        }
        .buttonStyle(.plain)
    }
    
    func image(for number: Int) -> Image {
        if number > rating {
            offImage ?? onImage
        } else {
            onImage
        }
    }
}

#Preview {
    RatingView(rating: .constant(4))
}

```

### Switches – Translate text to emojis

```swift
struct EmojiRatingView: View {
    let rating: Int
    var body: some View {
        switch rating {
            case 1: Text("😡")
            case 2: Text("😕")
            case 3: Text("🫤")
            case 4: Text("☺️")
            default: Text("🤩")
            
        }
    }
}

#Preview {
    EmojiRatingView(rating: 3)
}

```

### Decoding Data with Generics

### ContentView

```swift
import SwiftUI

struct ContentView: View {
    let astronauts: [String: Astronaut] = Bundle.main.decode("astronauts.json")
    let missions: [Mission] = Bundle.main.decode("missions.json")
    var body: some View {
        Text(String(astronauts.count))
    }
}
```

### Bundle-Decodable

```swift
import Foundation

extension Bundle {
    func decode<T: Codable>(_ file: String) -> T {
        guard let url = self.url(forResource: file, withExtension: nil) else {
            fatalError("Failed to locate \(file) in bundle.")
        }
        
        guard let data = try? Data(contentsOf: url) else{
            fatalError("Failed to load \(file) from bundle.")
        }
        
        let decoder = JSONDecoder()
        
        do {
            return try decoder.decode(T.self, from: data)
        } catch DecodingError.keyNotFound(let key, let context) {
            fatalError("Failed to decode \(file) from bundle due to missing key \(key.stringValue) – \(context.debugDescription)")
        } catch DecodingError.typeMismatch(_, let context) {
            fatalError("Failed to decode \(file) from bundle due to \(context.debugDescription)")
        } catch DecodingError.valueNotFound(let type, let context) {
            fatalError("Failed to decode \(file) from bundle due to missing \(type) value – ")
        } catch DecodingError.dataCorrupted(_) {
            fatalError("Failed to decode \(file) from bundle because it appears to be invalid JSON")
        } catch {
            fatalError("Failed to decode \(file) from bundle. \(error.localizedDescription)")
        }
    }
}

```

### Merging Codable Structs

Adding this to our view at bottom outside of body

```swift
init(mission: Mission, astronauts: [String: Astronaut]) {
        self.mission = mission
        
        self.crew = mission.crew.map { member in
            if let astronaut = astronauts[member.name] { //Here we find in the mission crew, the astronaut name
                return CrewMember(role: member.role, astronaut: astronaut)
            } else{
                fatalError("Missing \(member.name)")
            }
        }
    }
```

### Get rid of enum with class for sending data over the internet

```swift
enum CodingKeys: String, CodingKey {
        case _type = "type"
        case _quantity = "quantity"
        case _specialRequestEnabled = "specialRequestEnabled"
        case _extraFrosting = "extraFrosting"
        case _addSprinkles = "addSprinkles"
        case _name = "name"
        case _city = "city"
        case _streetAddress = "streetAddress"
        case _zip = "zip"
    }
```

### Setting up data across app with JSON

[Read this Moonshot project (link)](Day%2041%20cc9e8d0be8184f7ab7b95abd18ef85d6.md)

## Strings

### Print length of string

```swift
word.count //Where word is a string
```

### Remove character at position in string

```swift
tempWord.remove(at: pos) //where pos is 0, for example
```

### Lower casing and trimming characters

```python
newWord.lowercased().trimmingCharacters(in: .whitespacesAndNewlines)
```

### Check there’s more than one word in a string

```python
guard answer.count > 0 else {return}
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
    
    
func isReal(word: String) -> Bool {
        let checker = UITextChecker()
        let range = NSRange(location: 0, length: word.utf16.count)
        let mispelledRange = checker.rangeOfMisspelledWord(in: word, range: range, startingAt: 0, wrap: false, language: "en")
        
        return mispelledRange.location == NSNotFound
    }
```

### Checking if string can be made out of another one

```swift
func isPossible(word: String) -> Bool {
        var tempWord = rootWord
        for letter in word {
            if let pos = tempWord.firstIndex(of: letter) {
                tempWord.remove(at: pos)
            } else {
                return false
            } }
        return true }
```

### Copy text from List to clipboard

```swift
ForEach(emojis, id: \.self) {emoji in
                            Button(action: {
                                UIPasteboard.general.string = emoji
                            }, label: {
                                Text(emoji)
                            })
                           
```

## Dictionary

### Return Dictionaries (internal translation user choices to arrays)

```swift
var emojiDictionary: [String: [String]] {
        return [
            "Happy": happy,
            "Sad": sad,
            "Angry": angry,
            "Neutral": neutral,
            "Activities": activities,
            "Hand Expressions": hand_expressions,
            "Food": food
        ]
    }
   
var body: some View {
	if let emojis = emojiDictionary[selectedCategory] {
                        ForEach(emojis, id: \.self) {emoji in
                            Button(action: {
                                UIPasteboard.general.string = emoji
                            }, label: {
                                Text(emoji)
                            })
           }//:EMOJI DICTIONARY
  }
}
```

## Arrays

### Adding to an array at index

```python
usedWords.insert(answer, at:0)
```

### Find first index in a list

```swift
correctAnswer = options.firstIndex(of: correctResult) ?? 0 //Finds index of first correct anser for first time playing the game
```

### Shuffling/randomizing an array

```
let arr = [1,2,3].shuffled()
```

### Get Random element from array

```swift
let letter = letters.randomElement()
```

### Turn String into array

```swift
let letters = input.components(separatedBy: " ") 
```

## Functions

```
var exampleFunction: Double { ... } //Return type given as double
```

## Lists/Forms

### Forms

```
Form {
	Section { //We can say Section("Title here") {...} as well
		Text("Hello, world!")
		TextField("Enter your name", text: $name )
    Text("Your name is \\(name)")
	}
}
```

### NavigationStacks

```
NavigationStack{
            //Content
            .navigationTitle("SwiftUI")
            .navigationBarTitleDisplayMode(.inline)
}
```

### Create NavigationStack

```python
NavigationStack {
            NavigationLink {
                Text("Detail View")
            } label: {
                VStack {
                    Text("This is the label")
                    Text("So is this")
                }
                .font(.largeTitle)
            }
            .navigationTitle("SwiftUI")
```

### Creating a navigation stack with a list

```python
NavigationStack{
   List(0..<100) { row in
      NavigationLink("Row \(row)") {
        Text("Detail \(row)")
      }
   }
  .navigationTitle("SwiftUI")
}
```

### Adding navigation bar header color

```swift
struct ContentView: View {
    @State private var pathStore = PathStore() //Creating an array of integers
    var body: some View {
        NavigationStack {
            List(0..<100) { i in
                Text("Row \(i)")
            }
            .navigationTitle("Title goes here")
            .navigationBarTitleDisplayMode(.inline)
            .toolbarBackground(.blue)
            .toolbarColorScheme(.dark) //Adding this makes th text white
        }
    }
}
```

### Hiding the Navigation Bar

```swift
.toolbar(.hidden, for: .navigationBar) //Add this at the bottom after the other toolbar modifiers
```

### Disabled form entry

```swift
.disabled(order.hasValidAddress == false)
```

### Example Activities Form

```swift
struct AddActivityView: View {
    @State private var activityName = ""
    @State private var activityDescription = ""
    @State private var timeAmount = 0.0
    @State private var activityType = ""
    @Environment(\.dismiss) var dismiss
    
    let activityTypeChoices = ["Exercise", "Spiritual", "Goofing off", "Education", "Work", "Self care", "Travelling"]
    
    var activities: Activities
    var body: some View {
        NavigationStack {
            
            Form {
                
                    TextField("Activity Name", text: $activityName)
                        .font(.headline)
                    Section("Minutes spent") {
                        
                        Slider(value: $timeAmount, in: 0...120, step: 5)
                                                    .accentColor(.blue) // Customize the slider appearance
                            .keyboardType(.numberPad)
                        Text("\(Int(timeAmount)) minutes spent")
                             .foregroundColor(.secondary)
                    }
                    Picker("Select activity type", selection: $activityType) {
                        ForEach(activityTypeChoices, id: \.self) {
                            Text($0)
                        }
                    }
                        
                
                Section("Add a description") {
                    TextEditor(text: $activityDescription)
                        .frame(minWidth: 100, maxWidth: .infinity, minHeight: 100, maxHeight: 200)
                }
                    
                
                Button {
                    let item = ActivityItem(activityName: activityName, activityType: activityType, description: activityDescription, minutes: timeAmount)
                    activities.items.append(item)
                    dismiss()
                } label: {
                    Text("Save")
                        .frame(maxWidth: /*@START_MENU_TOKEN@*/.infinity/*@END_MENU_TOKEN@*/)
                }
                
            }
            
            .navigationTitle("Add an activity")
            
            
        }
    }
}
 
```

### ToolBarItem  –  Moving toolbar item location to leading edge (not trailing)

Placement here has .navigation .automatic .bottomBar .confirmationAction and lot's more options. These are all notably semantic.

```swift
NavigationStack {
            Text("Hello")
                .toolbar {
                    ToolbarItem(placement: .topBarLeading) {
                        Button("Tap Me") {
                            //action
                        }
                    }
                }
}
```

### Add Edit button to toolbar

```swift
.toolbar {
  ToolbarItem(placement: .topBarLeading) {
     EditButton()
  }
}
```

### ToolBarItemGroup  –  Grouping two buttons on toolbar

```swift
NavigationStack {
            Text("Hello")
                .toolbar {
                    ToolbarItemGroup(placement: .topBarLeading) {
                        Button("Tap Me") {
                            //action
                        }
                        
                        Button("Tap Me") {
                            //action
                        }
                    }
                }
}
```

### Add delete button from within a detail view

Add these modifiers to the top

```swift
@Environment(\.modelContext) var modelContext
    @Environment(\.dismiss) var dismiss
    @State private var showingDeleteAlert = false
```

Then add this after the .navigationTitle() under the scrollview

```swift
.scrollBounceBehavior(.basedOnSize)
.alert("Delete book", isPresented: $showingDeleteAlert) {
     Button("Delete", role: .destructive, action: deleteBook)
     Button("Cancel", role: .cancel) { }
  } message: {
    Text("Are you sure?")
  }
.toolbar {
    Button("Delete this book", systemImage: "trash") {
       showingDeleteAlert = true
    }
}
func deleteBook() {
        modelContext.delete(book)
        dismiss()
}
```

### Creating renamable NavigationStack

![Screenshot 2024-07-06 at 10.19.30.png](Screenshot_2024-07-06_at_10.19.30.png)

```swift
struct ContentView: View {
    @State private var title = "SwiftUI"
    var body: some View {
        NavigationStack {
            Text("Hello")
                .navigationTitle($title)
                .navigationBarTitleDisplayMode(.inline)
        }
    }
}
```

### ScrollView

### Create a horizontal ScrollView

```python
ScrollView(.horizontal) {
	Text("Hello")
}
```

### Make ScrollView take up full frame

```python
ScrollView {
  VStack(spacing: 10) {
    Text("H")
  }
}
.frame(maxWidth: .infinity) //Add this
```

### ScrollBounce Behavior

```swift
ScrollView {
//Content here
}
.scrollBounceBehavior(.basedOnSize
```

### Making a ScrollView load content on demand (with Lazy VStack)

This also works LazyHStack

```python
struct CustomText: View { //Just using this modifier to print out when the text is actually getting loaded
    let text: String
    
    var body: some View {
        Text(text)
    }
    
    init(text: String) {
        print("Creating a new CustomText")
        self.text = text
    }
}

struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVStack(spacing: 10) {
                ForEach(0..<100) {
                    CustomText(text: "Item \($0)")
                        .font(.title)
                }
            }
            .frame(maxWidth: .infinity)
        }
    }
}
```

### Lists

```swift
List {
	Text("Hello, world!")
	Text("Hello, world!")
	Text("Hello, world!")
}
```

### Date Pickers (with labels hidden)

```
Form {
   DatePicker("Please enter a time", selection: $wakeUp, displayedComponents: .hourAndMinute)
     .labelsHidden()
    } }

```

### Date formatting (short version, just day)

```swift
//17 July 2024
Text(book.current_date, format: .dateTime.year().month().day())

//17 July 2024 at 12:37:26 
Text(book.current_date.formatted(date: .abbreviated, time: .standard))
```

### Changing Forms Text to be plural/singular based on number of entries

```
Stepper("^[\\(coffeeAmount) cup](inflect: true)", value: $coffeeAmount, in: 1...20)
```

Note that the title goes on the inside.

### Disabling forms with validation

```swift
struct ContentView: View {
    @State private var username = ""
    @State private var email = ""
    var disableForm: Bool {
        username.count < 5 || email.count < 5
    }
    var body: some View {
        Form {
            Section {
                TextField("Username", text: $username)
                TextField("Email", text: $email)
            }
            Section {
                Button("Create account") {
                    print("Creating account")
                }
            }
            .disabled(username.isEmpty || email.isEmpty) 
            //OR .disabled(disableForm)
        }
    }
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

### Creating a list we can add to and delete from using the onDelete() modifier

```swift
import SwiftUI
struct ContentView: View {
    @State private var numbers = [Int]()
    @State private var currentNumber = 1
    
    var body: some View {
        NavigationStack {
            VStack {
                List {
                    ForEach(numbers, id: \.self) {
                        Text("Row \($0)")
                    }
                    .onDelete(perform: removeRows)
                }
                Button("Add number") {
                    numbers.append(currentNumber)
                    currentNumber += 1
                }
            }
            .navigationTitle("iExpense")
            .toolbar {
                EditButton()
            }
        }
    }
    
    func removeRows(at offsets: IndexSet) {
        numbers.remove(atOffsets: offsets)
    }
}
```

### Looping over an array in ListView

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

### Conditional modifiers

```
let age = 18
let canVote = age >= 18 ? "Yes" : "No"
```

### Environment Modifiers versus Regular Modifiers

This will create a standard body font but then also create an exception for Gryffindor that is large title.

```
VStack {
   [ Text("Gryffindor")
     .font(.largeTitle)
     Text("Ravenclaw")]
}
.font(.body)
```

### Using text as a variable

```
let motto1 = Text("Hello there")
let motto2 = Text("Hello there 2")
//Other option
@ViewBuilder var spells: some View {
	Text("Hello there")
	Text("Hello there 2")
}
var body: some View {
	VStack {
		motto1
			.foregroundStyle(.red)
		motto2
			.foregroundStyle(.blue)
	}
}
```

### Reducing text modifers to one struct (to remove duplicates)

```
struct CapsuleText: View {
    var text: String
    var body: some View {
        Text(text)
            .foregroundColor(.blue)
    } }
struct ContentView: View {
    var body: some View {
        CapsuleText(text: "First")
    } }
```

### Create custom modifer for text

```

struct Title: ViewModifier {
    func body(content: Content) -> some View {
        [content
          .font(.largeTitle)
          .foregroundColor(.white)
          .padding()
          .background(.blue)
          .clipShape(.rect(cornerRadius: ]10))
    } }
struct ContentView: View {
    var body: some View {
        Text("Hello, world")
            .modifier(Title())
    } }
```

### Combining view modifiers with extensions to View

Custom view modifiers can have their own stored properties such as var text: String but extensions to view cannot.

```
struct Watermark: ViewModifier {
    var text: String

    func body(content: Content) -> some View {
        ZStack(alignment: .bottomTrailing) {
            content
            Text(text)
                .font(.caption)
                .foregroundStyle(.white)
                .padding(5)
                .background(.black)
        }
    } }

extension View {
    func watermarked(with text: String) -> some View {
        modifier(Watermark(text: text))
    }
}

struct ContentView: View {
    var body: some View {
        Color.blue
            .frame(width: 300, height: 200)
            .watermarked(with: "Hacking with Swift")
    } }
```

### Another example of Structs and extensions

```swift
struct CornerRotateModifier: ViewModifier {
    let amount: Double
    let anchor: UnitPoint
    
    func body(content: Content) -> some View {
        content
            .rotationEffect(.degrees(amount), anchor: anchor)
            .clipped()
    }
}

extension AnyTransition {
    static var pivot: AnyTransition {
        .modifier(active: CornerRotateModifier(amount: -90, anchor: .topLeading), identity: CornerRotateModifier(amount: 0, anchor: .topLeading))
    }
}
```

### Two-way binding ($)

We need to tell Swift to read and write the value automatically. Changing string means to update the page etc.

Just writing name means to just read the value. But the dollar sign means to show the name here but also update when we change it here.

Example:

```
@Stateprivate vartapCount = 0
varbody: someView {
	TextField("Enter your name", text: $name )
	Text("Your name is \\(name)")
}
```

## Math and Numbers

### Create random number

```
var correctAnswer = Int.random(in: 0...2)
```

## Shapes

### Square that changes into corned square and back

```swift
Button("Tap Me") {
            enabled.toggle()
        }
        
        .frame(width: 200, height: 200)
        .background(enabled ? .blue : .red)
        .foregroundStyle(.white)
        .clipShape(.rect(cornerRadius: enabled ? 60 : 0))
        .animation(.default, value: enabled)
```

### Rectangle middle of screen

```swift
LinearGradient(colors: [.yellow,.red], startPoint: .topLeading, endPoint: .bottomTrailing)
            .frame(width: 300, height: 200)
            .clipShape(.rect(cornerRadius: 10))
```

## Working with Dates + Date Pickers

### Creating Date (with Time and Interval) and Start and End of Day

```
let tomorrow = Date.now.addingTimeInterval(86400)
        let range = Date.now...tomorrow
let startOfDay = Calendar.current.startOfDay(for: Date())
let endOfDay = Date()
```

### Date formatting

```
Text(Date.now, format:.dateTime.hour().minute) Text(Date.now, format: .dateTime.hour().month().year()) Text(Date.now.formatted(date: .long, time: .shortened))

```

### Creating Default Time

```
static var defaultWakeTime: Date {
        var components = DateComponents()
        components.hour = 7
        components.minute = 0
        return Calendar.current.date(from: components) ?? .now
    } //Static var, so belongs to contentview, not just an instance.
```

### Date Pickers

```
//Date picker with the date
@State private var wakeUp = Date.now
var body: some View {
 	DatePicker("Please enter a date", 	selection: $wakeUp)
}
.labelsHidden() //Hide date picker label

//Date picker with only the time
DatePicker("Please enter a date", selection: $wakeUp, displayedComponents: .hourAndMinute)
```

## Pickers

### Name Picker

```
let students = ["harry", "hermione", "ron"]
    @State private var selectedStudent = "Harry"
    var body: some View {
		Form {
  			Picker("Select your student", selection: 		$selectedStudent) {
   				 ForEach(students, id: \.self) {
   					Text($0)
    			  } } } }
```

id: \.self *is telling SwiftUI that each string itself is unique – how to break it downvf.

### Number Picker (Optional open new page)

```
struct ContentView: View {
    @State private var checkAmount = 0.0
    @State private var numberOfPeople = 2
	var body: some View {
       //NavigationStack{
		Form {
            Section {
				Picker("Number of people", selection: $numberOfPeople) {
                    ForEach(2..<100) {
                        Text("\\($0) people")
                    }
                }//.pickerStyle(.navigationLink) //Relies on being in 		navigaitonstack as shown
            }
			}
		//}

```

### Pickers with tags

```swift
Picker("Sort", selection: $sortOrder) {
                    Text("Sort by name")
                        .tag([
                            SortDescriptor(\User.name),
                            SortDescriptor(\User.joinDate)
                        ])
                    Text("Sort by join date")
                        .tag([
                            SortDescriptor(\User.joinDate),
                            SortDescriptor(\User.name)
                        ])
                }
```

### Picker Style options

```
.pickerStyle(.segmented)
.pickerStyle(.navigationLink) //Relies on whole form/picker being in navigationStack
```

### Slider

```swift
Slider(value: $timeAmount, in: 0...200, step: 5)
   .accentColor(.blue)
```

### Stepper

```swift
Stepper("\\(sleepAmount.formatted()) hours", value: $sleepAmount, in: 4...12, step: 0.25);
//Ranges 4-12 inclusive, steps by 0.25, the .formatted() removes the excess data)
```

## Textfields

### Asking for a number form the user in textfield

```
Form {
   Section {
      TextField("Amount", value: $checkAmount, format: .currency(code: "USD")) //OR (code: Locale.current.currency?.identifier ?? "USD")) //Defaults to USD
          }
  }
```

We ask swift system for the currency set on device

### OnSubmit Function when Text Entered

```python
List {TextField("Enter your word", text: $newWord)}
.onSubmit(addNewWord)
```

### Disable capitalization in textfield

```python
//textfield Here
.textInputAutocapitalization(.never)
```

### Bring up number keyboard instead (Force number keyboard)

```
TextField("Amount", value: $checkAmount, format: .currency(code: Locale.current.currency?.identifier ?? "USD"))
 .keyboardType(.decimalPad) //This keyboard type goes after the text field

```

### Dismiss the text keyboard

```
struct ContentView: View {
	@FocusState private var amountIsFocused: Bool //New type for keeping track of focus
	var body: some View {
	Form {
	 TextField("Amount", value: $checkAmount)
		.focused($amountIsFocused)
	}
	}
	.toolbar {
       if amountIsFocused {
       	Button("Done") {
        	amountIsFocused = false
        } //:BUTTON
       }
    }//:TOOLBAR
}

```

### TextEditor or Textfield with multiple lines allowed

```swift
Section("Add a description") {
                    TextEditor(text: $activityDescription)
                        .frame(minWidth: 100, maxWidth: .infinity, minHeight: 100, maxHeight: 200)}
```

## Button

### General

```
Button("Delete selection") {
    print("Now deleting")
}Button("edit", systemImage: "pencil") {
                print("Tapped")
}

```

### Create round button

```swift
 Button("Tap me") {
            //Do nothing
        }
        .padding(50)
        .background(.red)
        .foregroundStyle(.white)
        .clipShape(.circle)
```

### Telling SwiftUI it's Dangerous Button

```
Button("Delete selection", role: .destructive) {
            print("Now deleting")
        }

```

### Adding Button Label Views (Instead of HStacks)

```
Button(action: {
            print("")
        }, label: {
            Label("Edit", systemImage: "pencil")
        })

```

### Button Bordering options

```
Button("Button 1") {}
    .buttonStyle(.bordered)
       Button("Button 2", role: .destructive) {}
                .buttonStyle(.bordered)
       Button("Button 3") {}
                .buttonStyle(.borderedProminent)
       Button("Button 4", role: .destructive) {}
                .buttonStyle(.borderedProminent)
                .tint(.indigo)]

```

### Custom Button

```
Button { print("Button was tapped")}
		label: {
                Text("Tap me")
                    .padding()
                    .foregroundStyle(.white)
                    .background(.red)
            }

```

### Creating ClipShape + Adding Shadow

```
Button {
	flagTapped(number)
} label: {
	Image(countries[number])
      .clipShape(.capsule)
	  .shadow(radius: 5)
}

```

### Center text within a button

```swift
Button {
 let item = ActivityItem(activityName: activityName, activityType: activityType, description: activityDescription, minutes: timeAmount)
	activities.items.append(item)
  dismiss()
 } label: {
Text("Save")
    .frame(maxWidth: /*@START_MENU_TOKEN@*/.infinity/*@END_MENU_TOKEN@*/)
}
```

### Button adjusting key variables

This example is useful because it is faster than creating and destroying two different buttons, one blue and one red.

```
@State private var useRedText = false
Button("Tap here") {
	useRedText.toggle())
}
.foregroundStyle(useRedText ? .red : .blue)

```

## Showing Alerts

### Simple alerts

```
struct ContentView: View {
    @State private var showingAlert = false
    var body: some View {
        Button("Create alert") {
            showingAlert = true
        }
        .alert("Important message", isPresented: $showingAlert) {
            Button("OK") {}
        }
    }
}

```

### Making more sophisticated multi-button alerts

```
struct ContentView: View {
    @State private var showingAlert = false
    var body: some View {
        Button("Create alert") {
            showingAlert = true
        }
        .alert("Important message", isPresented: $showingAlert) {
            Button("Delete", role: .destructive) {}
            Button("Delete", role: .cancel)     {}
        } message: {
            Text("Please Read this")
        }//Dismisses the alert, can add more, but no matter what ends with dismissing
    }
}

```

## Images

### Getting iOS not to Voice Over Images

```
image(decorative: "exampleName")
```

### SystemName Images

```
Image(systemName: "pencil.circle")
```

### Naming files for different scales

If we have a picture called `Italy` , we can just call the different scales `Italy@2xn and Italy@3`x

## Colors

### Adding Text background

```
ZStack {
            Text("Your content")
        }
        .background(.red)

```

### Adding Full Page Background color

```
ZStack {
            Color.red //Here Color.red is its own view
            Text("Your content")
        }

```

### Semantic Colors

```
Color.primary
Color.secondary

```

### Materials/Adding Frosted Glass

```
.ultraThinMaterial
.ultraThickMaterial
.materialDark
.materialLight
.material
```

### Add color theming extension (added as separate file)

```swift
import SwiftUI

extension ShapeStyle where Self == Color {
    static var darkBackground: Color {
        Color(red: 0.1, green: 0.1, blue: 0.2)
    }
    
    static var lightBackground: Color {
        Color(red: 0.2, green: 0.2, blue: 0.3)
    }
}
```

### Make screen split into two colors

```
ZStack {
            VStack {
                Color.red
                Color.blue
            }
        }
        .ignoresSafeArea()

```

## Color Gradients

### Simple Gradient

```
var body: some View {
        LinearGradient(colors: [.white, .black], startPoint: .top, endPoint: .bottom)
    }

```

### Adding multiple stops (harsher transition)

```
LinearGradient(stops: [
            Gradient.Stop(color: .white, location: 0.45),
            Gradient.Stop(color: .black, location: 0.55)],
            startPoint: .top, endPoint: .bottom)

```

```
LinearGradient(stops: [
            .init(color: .white, location: 0.45),
            .init(color: .black, location: 0.55)],
            startPoint: .top, endPoint: .bottom)

```

### Angular Gradient

```
AngularGradient(colors: [.blue, .yellow, .green, .blue, .purple, .red], center: .center)

```

### Very simple + subtle gradient

```
Text("Your content")
            .frame(maxWidth: .infinity, maxHeight: .infinity)
            .foregroundStyle(.white)
            .background(.red.gradient)

```

### Radial Gradient

```
RadialGradient(colors: [.blue, .black], center: .center, startRadius: 20, endRadius: 200)

```

## Modifiers and Stacks

### Sizing in View

```
View()
	.frame(minWidth: 200, maxWidth: .infinity, maxHeight:  200)
	.ignoreSafeArea() //Make content ignore safe areas
	
	Spacer() //Adding a spacer
```

### Divide Screen into Two Covered Colors

```
RadialGradient(stops: [
                .init(color: .blue, location: 0.3),
                .init(color: .red, location: 0.3)], center: .top, startRadius: 200, endRadius: 700
```

### Adding alignment/spacing to stacks

Other alignments include .top, .leading, .trailing

```
 VStack(alignment: .leading, spacing: 3) {
          Text("Hello")
        }
```

### Creating a 3x3 Grid

```
HStack {
    ForEach(0..<3) { _ in
        VStack {
            ForEach(0..<3) { _ in
                HStack {
                    Text("H")
                } } } } }

```

### Alternative Grid Layout (using built-in method)

### Creating Grid layout

```python
let layout =  [
        GridItem(.fixed(80)),
        GridItem(.fixed(80)),
        GridItem(.fixed(80))
        ]
```

### Creating adapative grid layout

```python
struct ContentView: View {
    let layout =  [ GridItem(.adaptive(minimum: 80)) ]
    var body: some View {
        ScrollView(.horizontal){
            LazyHGrid(rows: layout) {
                ForEach(0..<1000) {
                    Text("Item \($0)")
                }
            }
} } }
```

### Showing sheets and using views declared elsewherre

```swift
struct SecondView: View {
    @Environment(\.dismiss) var dismiss
    let name: String
    var body: some View {
        Text("Hello, \(name)")
        Button("Dismiss") {
            dismiss()
        }
    }
}

struct ContentView: View {
    @State private var showingSheet = false
    
    var body: some View {
        Button("Show sheet") {
            showingSheet.toggle()
        }
        .sheet(isPresented: $showingSheet) {
            SecondView(name: "Sam")
        }
    }
}
```

### Dismissing the sheet

Add this to the sheet view in the top

```python
@Environment(\.dismiss) var dismiss
```

Then for the “Save” or “Close” button action within the sheet add this dismiss()

```python
.toolbar {
                Button("Save") {
                    let item = ExpenseItem(name: name, type: type, amount: amount)
                    expenses.items.append(item)
                    dismiss()
                }
            }
```

## Animation

### Animation for a button

```swift
Button("Tap me") {
            animationAmount += 1.0
}
.scaleEffect(animationAmount)
.blur(radius: (animationAmount - 1) * 3)
.animation(.default, value: animationAmount)

//Other options
.animation(.linear, value: animationAmount) //for linear transformation
.animation(.spring(duration: 1, bounce: 0.9), value: animationAmount) //Bouncier
.animation(.easeInOut(duration: 2), value: animationAmount) //EaseInOut

//More minutae
.animation(
            .easeInOut(duration: 2)
            .delay(1), //.repeatCount(2, autoreverses: true),
            .repeatForever(autoreverses: true),
            value: animationAmount
        )
```

### Animation for inserting into an array

```
withAnimation { 
  usedWords.insert(answer, at:0)
}
```

### Spinning Button

```swift

struct ContentView: View {
    @State private var animationAmount = 0.0
    var body: some View {
        Button("Tap Me") {
            withAnimation(.spring(duration: 1, bounce: 0.9)) {
                animationAmount += 360
        } }
        .padding(50)
        .background(.red)
        .foregroundStyle(.white)
        .clipShape(.circle)
        .rotation3DEffect(
            .degrees(animationAmount), axis: (x: 1.0, y: 1.0, z: 0.0))
    } }
```

### Concentric Circles from Button

Note: Relies on @State **private** **var** animationAmount = 0.0 set up

```swift
 Button("Tap me") { // }
        .padding(50)
        .background(.red)
        .foregroundStyle(.white)
        .clipShape(.circle)
        .overlay(Circle()
            .stroke(.red)
            .scaleEffect(animationAmount)
            .opacity(2 - animationAmount)
            .animation(
                .easeOut(duration: 2)
                .repeatForever(autoreverses: false),
                value: animationAmount
            )
        )
        .onAppear {
            animationAmount = 2
        }
```

### Animating appearing/disappearing elements from screen

```swift
import SwiftUI

struct ContentView: View {
    
    @State private var isShowingRed = false
    var body: some View {
        VStack {
            Button("Tap Me") {
                withAnimation {
                    isShowingRed.toggle()
                }
            }
            if isShowingRed {
                Rectangle()
                    .fill(.red)
                    .frame(width: 200, height: 200)
                    .transition(.scale)
                    /*We can add more granularity with
                    .transition(.asymmetric(insertion: .scale, removal: .opacity)) */
            } } } }
```

## Haptics

### Haptic feedback from @State variable change

```swift
struct ContentView: View {
    @State private var counter = 0
    var body: some View {
        Button("Tap Count: \(counter)") {
            counter += 1
        }
        .sensoryFeedback(.increase, trigger: counter)
		  //.sensoryFeedback(.impact(flexibility: .soft, intensity: 0.5), trigger: counter)
    }
}
```

### General guide

```swift
import CoreHaptics
import SwiftUI

struct ContentView: View {
    @State private var engine: CHHapticEngine?
    var body: some View {
        Button("Play haptic") {
            complexSuccess()
        }
        .onAppear(perform: prepareHaptics)
    }
    
    func prepareHaptics() {
        guard CHHapticEngine.capabilitiesForHardware().supportsHaptics else { return }
        do {
            engine = try CHHapticEngine()
            try engine?.start()
        } catch {
            print("There was an error creating the engine. \(error.localizedDescription)")
        }
    }
    
    func complexSuccess() {
        guard CHHapticEngine.capabilitiesForHardware().supportsHaptics else { return }
        var events = [CHHapticEvent]()
        
        let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: 1)
        let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: 1)
        let event = CHHapticEvent(eventType: .hapticTransient, parameters: [intensity, sharpness], relativeTime: 0)
        events.append(event)
        
        do {
            let pattern = try CHHapticPattern(events: events, parameters: [])
            let player = try engine?.makePlayer(with: pattern)
            try player?.start(atTime: 0)
        } catch {
            print("Failed to play pattern with \(error.localizedDescription)")
        }
    }
}
```

## Gestures

### Drag Box around screen

```swift
import SwiftUI
struct ContentView: View {
    @State private var dragAmount = CGSize.zero
    var body: some View {
        LinearGradient(colors: [.yellow,.red], startPoint: .topLeading, endPoint: .bottomTrailing)
            .frame(width: 300, height: 200)
            .clipShape(.rect(cornerRadius: 10))
            .offset(dragAmount)
            .gesture(
                DragGesture()
                    .onChanged { dragAmount = $0.translation }
                    .onEnded {_ in dragAmount = .zero}
                )
    } }
```

### Add bounciness to dragging

We add .animation(.bouncy, value: dragAmount) after gesture or we can do the following

```swift

import SwiftUI

struct ContentView: View {
    @State private var dragAmount = CGSize.zero
    var body: some View {
        LinearGradient(colors: [.yellow,.red], startPoint: .topLeading, endPoint: .bottomTrailing)
            .frame(width: 300, height: 200)
            .clipShape(.rect(cornerRadius: 10))
            .offset(dragAmount)
            .gesture(
                DragGesture()
                    .onChanged { dragAmount = $0.translation }
                    .onEnded {_ in
                        withAnimation(.bouncy) {
                            dragAmount = .zero
                        }
                    }
                )
            .animation(.bouncy, value: dragAmount)
    }
}
```

### Making cool animations of an array like a snake

```swift
import SwiftUI

struct ContentView: View {
    let letters = Array("Hello SwiftUI")
    @State private var dragAmount = CGSize.zero
    @State private var enabled = false
    var body: some View {
        HStack(spacing: 0) {
            ForEach(0..<letters.count, id: \.self) { num in
                Text(String(letters[num]))
                    .padding(5)
                    .font(.title)
                    .background(enabled ? .blue : .red)
                    .offset(dragAmount)
                    .animation(.linear.delay(Double(num) / 20), value: dragAmount)
            }
        }
        .gesture (
            DragGesture()
                .onChanged {dragAmount = $0.translation}
                .onEnded { _ in
                    dragAmount = .zero
                    enabled.toggle()
                }
        )
    }
}
```

## Networking and Asynchronous functions

Synchronous functions work straight away. An asynchronous functions can go to sleep for a while and wait for work to complete. This is useful for coding.

## Networking and asynchronous functions

### Create asynchronous function (async)

```swift
func loadData() async {
        
}
```

### Create URL to fetch

```swift

struct Response: Codable {
    var results: [Result]
}

struct Result: Codable {
    var trackId: Int
    var trackName: String
    var collectionName: String
}

struct ContentView: View {
    @State private var results = [Result]()
    
    var body: some View {
        List(results, id: \.trackId) { item in
            Text(item.trackName)
                .font(.headline)
            
            Text(item.collectionName)
                
        }
        .task {
            await loadData()
        }
    }
    
    func loadData() async {
        //Create URL we want to read from Apple's servers
        guard let url = URL(string: "https://itunes.apple.com/search?term=taylor+swift&entity=song") else {
            print("Invalid URL")
            return
        }
        
        //Fetch data from the URL using Swift
        do {
            let (data, _) = try await URLSession.shared.data(from: url) //URLSession returns data and metadata about that request. We just keep the finished data and discard the response
            
            //Convert data object into response object using JSONEncoder and assign to results array
            if let decodedResponse = try? JSONDecoder().decode(Response.self, from: data) {
                results = decodedResponse.results
            }
        } catch {
            print("Invalid data")
        }
    }
}
```

### Quickly fetch an image with Async and resize it

SwiftUI loads the image and displays it at full size. We can tell the image as it’s downloading it’s a 3x image. We can’t just use .reszable() and .frame(). 

```swift
var body: some View {
   AsyncImage(url: URL(string: "https://hws.dev/img/logo.png"))
   AsyncImage(url: URL(string: "https://hws.dev/img/logo.png"), scale: 3) //Scale option
   
}

//Option 1 for resizing
var body: some View {
        AsyncImage(url: URL(string: "https://hws.dev/img/logo.png")) { image in
            image
                .resizable()
                .scaledToFit()
        } placeholder: {
            ProgressView()
        }
        .frame(width: 200, height: 200) 
    }
   
//Option 2 for resizing
AsyncImage(url: URL(string: "https://hws.dev/img/logo.png")) { phase in
     if let image = phase.image {
        image.resizable()
          .scaledToFit()
     } else if phase.error != nil {
        Text("There was an error loading the image")
     } else {
        ProgressView()
    }
}
.frame(width: 200, height: 200)

```

### Add Progress view spinner

```swift
ProgressView()
```

# Other

## Networking

### Accessing Website for networking prototyping

This function should be called with this button:

```swift
Button("Place Order") {
     Task {
        await placeOrder()
     }
}
```

### **The placeOrder() function**

```swift
func placeOrder() async {
        guard let encoded = try? JSONEncoder().encode(order) else {
            print("Failed to encode order")
            return
        }
        let url = URL(string: "https://reqres.in/api/cupcakes")!
        var request = URLRequest(url: url)
        request.setValue("application/json",forHTTPHeaderField: "Content-Type")
        request.httpMethod = "POST"
        
        do {
            let (data, _) = try await URLSession.shared.upload(for: request, from: encoded)
            
            //Read result here
            let decodedOrder = try JSONDecoder().decode(Order.self, from: data)
            confirmationMessage = "Your order for \(decodedOrder.quantity)x \(Order.types[decodedOrder.type].lowercased()) cupcakes is on its way!"
            print(confirmationMessage)
            showingConfirmation = true
            
        } catch {
            print("Check out failed: \(error.localizedDescription)")
        }
    }
```

### Debugging networking function to see data

Add a break point at line where let url is created  and use the simulator to get to point where that line is used. And then use the green thing at the bottom of XCode “lldb” and enter this to print out our encoded data

```swift
p String(decoding: encoded, as: UTF8.self)
```

## SwiftData

### Sort SwiftUI Data by alphabetical order of the struct “title” field

Change the @Query var books: [Book] to this:

```swift
@Query(sort: \Book.title) var books: [Book]
```

We can also do this for reverse:

```swift
@Query(sort: [SortDescriptor(\Book.title, order: .reverse)]) var books: [Book]
```

### Adding two sorting options to @Query

```swift
@Query(sort: [
        SortDescriptor(\Book.title, order: .reverse),
        SortDescriptor(\Book.author, order: .reverse),
    ]) var books: [Book]
```

### Add on delete with SwiftData to a list

```swift
NavigationStack {
            List {
                ForEach(books) { book in
                    NavigationLink(value: book) {
                        HStack {
                            EmojiRatingView(rating: book.rating)
                                .font(.largeTitle)
                            VStack(alignment: .leading){
                                Text(book.title)
                                    .font(.headline)
                                
                                Text(book.author)
                                    .foregroundStyle(.secondary)
                            }
                        }
                    }
                }
                .onDelete(perform: deleteBooks)
            }
            .navigationTitle("Bookworm")
```

**Then below in the struct add this function:**

```swift
func deleteBooks(at offsets: IndexSet) {
        for offset in offsets {
            let book = books[offset]
            modelContext.delete(book)
        }
    }
```

## HealthKit

[Authenticate HealthKit Access](Authenticate%20HealthKit%20Access%20cd6c531c8d3c423e990f6f1aa68384bd.md)

[Get Healthkit Data](Get%20Healthkit%20Data%20d5cc0c447ccd4b25a7c0690ed2780116.md)

## CreateML

Put in Training data, for target we put what we try to predict (e.g. actual sleep) and then for features we use what we try to do for predicting the output (e.g., was coffee drunk before). We can then select the algorithm we want to train our data. Then we select Train and then go to "training"

Then on evaluation page, it tells you under "Validation" and then "Root Value Square Error" what the actual error rate was.

The actual model here will be 100 bytes, only relationship between values.

## Getting Location Access

### Temporary Location Access

```swift
import CoreLocation
import CoreLocationUI

class LocationManager: NSObject, ObservableObject, CLLocationManagerDelegate {
    let manager = CLLocationManager()
    @Published var location: CLLocationCoordinate2D?
    
    override init() {
        super.init()
        manager.delegate = self
        manager.requestWhenInUseAuthorization()
    }
    
    func requestLocation() {
        manager.requestLocation()
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        location = locations.first?.coordinate
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("Failed to find user's location: \(error.localizedDescription)")
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        switch manager.authorizationStatus {
        case .authorizedWhenInUse, .authorizedAlways:
            manager.requestLocation()
        case .denied, .restricted:
            print("Location access denied or restricted.")
        case .notDetermined:
            manager.requestWhenInUseAuthorization()
        @unknown default:
            print("Unknown location authorization status.")
        }
    }
}

struct ContentView: View {
	@StateObject var locationManager = LocationManager()
var body: some View {
	VStack {
            if let location = locationManager.location {
                Text("Your location: \(location.latitude), \(location.longitude)")
            }
            LocationButton(.currentLocation) {
                locationManager.requestLocation()
            }
            .padding()
      }
```

### Permanent Location Access

**Important:** Enable this in PList

![Screenshot 2024-06-21 at 00.08.30.png](Screenshot_2024-06-21_at_00.08.30.png)

```swift
import SwiftUI
import CoreLocation

class LocationManager: NSObject, ObservableObject, CLLocationManagerDelegate {
    private var locationManager = CLLocationManager()
    
    @Published var location: CLLocation?
    
    override init() {
        super.init()
        self.locationManager.delegate = self
        self.locationManager.desiredAccuracy = kCLLocationAccuracyBest
        self.locationManager.requestWhenInUseAuthorization()
    }
    
    func startUpdating() {
        locationManager.startUpdatingLocation()
    }
    
    func stopUpdating() {
        locationManager.stopUpdatingLocation()
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        self.location = locations.last
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("Failed to get location: \(error.localizedDescription)")
    }
}

struct ContentView: View {
    @StateObject private var locationManager = LocationManager()
    let healthStore = HKHealthStore()
    
    var body: some View {
        VStack {
            Button("Activate") {
                activateHealth()
            }
            Button("Get Time In Daylight") {
                fetchTimeInDaylightPerHour()
            }
            if let location = locationManager.location {
                Text("Location: \(location.coordinate.latitude), \(location.coordinate.longitude)")
                    .padding()
            } else {
                Text("Location not available")
                    .padding()
            }
        }
        .onAppear {
            locationManager.startUpdating()
        }
    }
   }
```

# Recording Days of SwiftUI Code

[SwiftUI Day 16](notion://www.notion.so/swiftui-day-16.md)

[SwiftUI Day 17](notion://www.notion.so/swiftui-day-17.md)

[SwiftUI Day 20](notion://www.notion.so/swiftui-day-20.md)

[Day 26](notion://www.notion.so/day-26.md)

[Day 29](Day%2029%20b5c0cdc02dd74d95ad5a27d037cacacf.md)

[Day 30](Day%2030%209daa906376524058b0b6072ac4593109.md)

[Day 32](Day%2032%20ab0617b22a7c483e8490a3d893c0489c.md)

[Day 33](Day%2033%20aabbaec206fc46f2bfc4630fb3643edc.md)

[Day 36](Day%2036%2025f3b4f2f75c413e8838f597214fb5ba.md)

[Day 37](Day%2037%206ba9adb4d10148d8a266b28332cfb31c.md)

[Day 39](Day%2039%2063698d218a7344c5b329fcde7a7d04b7.md)

[Day 40](Day%2040%207055c3e3991344a9bb1165837e1cd65c.md)

[Day 41](Day%2041%20cc9e8d0be8184f7ab7b95abd18ef85d6.md)

[Days 43 + 44](Days%2043%20+%2044%20d0889455927b47bea6ef3dd3943bb790.md)

[Day 45](Day%2045%203e6af23a49e5471ea6b3388a5a3cca4d.md)

[Day 49](Day%2049%204dedb6f5bae94684a81145b7fb3d4a30.md)

[Day 50](Day%2050%202c3a25eaa34e4ad7a27e663379c4ae58.md)

[Day 51](Day%2051%200a85c1d419ec49b7b81d6c8bfb11b0fd.md)

[Day 53](Day%2053%20bec571c3d2d1484883210369093e98ad.md)

[Day 54](Day%2054%204e3cea6ae4a3493b991349a7e5017625.md)

[Day 55](Day%2055%20c1417687944e4f43bcefe0d867487e06.md)

[Day 57](Day%2057%203981b0969f2d41438a2f0b9493010661.md)

[Day 58](Day%2058%202efb24a783204866876411aa4fe75af1.md)

[Day 62](Day%2062%20cf614a87c43f4cc2a65f8de7e6cd75f6.md)

[Day 63](Day%2063%20eda9215479744b368380cdb776237a64.md)

[Day 64](Day%2064%2054a358d60e9a403db4a9f25d9cab3415.md)

[Day 65](Day%2065%20dfee42bb07cd48bb9495b11d856d170f.md)

[Day 66](Day%2066%206c068268554949eebf008d5acf470459.md)

[Day 68](Day%2068%20799a1454ee3e40a3827113f841bfeee1.md)

[Day 69](Day%2069%200a0c76edeca94a5a91d00af6a54361a6.md)

[Day 70](Day%2070%20a4bd6c6f77c949459e0f61fd51269c53.md)