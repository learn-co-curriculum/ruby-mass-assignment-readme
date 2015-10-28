---
tags:
level:
languages:
resources:
---

# Ruby Mass Assignment

We know objects can have properties and that we can assign those properties values on initialization by defining initialize with arguments for each property. That works great for 1-3 properties.

Person.new("Avi", "Male", "31")

It's clear what each argument is, name, gender, age. But look what happens when we have more than 3 properties.

Person.new("Avi", "Male", "31", "Programmer", "California", "New York", "Brown", "170")

Suddenly the meaning of each argument and value is lost. We'd have to build something like

initialize(name, gender, age, occupation, current_state, home_state, eye_color, weight)

And then we'd have to manually assign each argument to the corresponding instance variable.

And we'd have to remember the order of each argument everytime we make a person.

Imagine the following

Person.new({
  name: "Avi",
  gender: "Male",
  age: "31",
  occupation: "Programmer",
  current_state: "California",
  home_state: "New York",
  eye_color: "Brown",
  weight: "170"
})

By passing a hash of properties to initialize, the metadata, the semantics of each value, is present. We know "170" is the weight and "California" is the current_state.

Additionally, the order of the arguments dont matter anymore because keys and values in a hash aren't ordered.

Person.new({
  weight: "170",
  gender: "Male",
  name: "Avi",
  current_state: "California",
  age: "31",
  occupation: "Programmer",
  eye_color: "Brown",
  home_state: "New York"
})

That would still be passing only one argument to new, the property hash.

But how do we implement that in initialize

initialize(properties)
  @name = properties[:name]
  etc
end

we could hard code it, but then everytime a person has a new property, we have to update initialize.

Meta programming

Meta programming as a concept

what we'd want to do in initialize psuedocode

for each key in properties, find the instance variable corresponding to that key and assign the key's value to the instance variable's value.

Could we somehow use the character values of the keys in the hash, like :name, to somehow call the name= method on the person and then assign the value?

initialize
  self.name = properties[:name]
end

that's what we're trying to accomplish literally but instead of hard coding the name, we want to use keys in the hash. imagine the following (which won't work)

initialize
  properties.each do |property_name, property_value|
  self.property_name = property_value

end

We'd be trying to get ruby to literally translate that code into something like

self.name = property_value

but instead ruby will take the code literally and instead of using the value of the property_name variable, ruby will think we're actually trying to call property_name = which doesn't exist.

we could try something like

initialize
  properties.each do |property_name, property_value|
  self."#{property_name} =" property_value

end


But interpolation of strings doesn't work like that - conceptually it's right - use the property_name to call the corresponding attribute writer. we just need to learn about a magical method called send.

let's look at a simpler example

method_to_call = "reverse"
"reverse this string".method_to_call

here again, ruby will take method_to_call literally, not as a variable but as a method call sent to the string "reverse this string".

method_to_call = "reverse"
"reverse this string"."#{method_to_call}"

also doesn't work.

method_to_call = "reverse"
"reverse this string".send(method_to_call)

the send method is available to all objects and is used to delegate a method call on the object.

"reverse me".send("reverse") is the same as saying "reverse me".reverse - instead of calling reverse directly on the object, we're asking send to call the reverse method based on the string value sent to it.

with this we can metaprogram mass Assignment

full example

explain send("#{property_name}=", property_value)
where the 2nd argument is the argument passed to the method

this is called dynamic dispatch and is a key technique in metaprogramming.
