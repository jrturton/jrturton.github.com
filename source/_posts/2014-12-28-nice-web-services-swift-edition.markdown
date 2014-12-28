---
layout: post
title: "Nice Web Services, Swift edition"
date: 2014-12-28 21:03:29 +0000
comments: false
categories: 
---

As I've [written before](http://commandshift.co.uk/blog/2014/01/02/nice-web-services/) I am very keen on getting web service results out of JSON and into a strongly typed form as soon as possible. It makes the rest of your code much cleaner, insulates you from knowledge of the web service keys and format and aids testing. 

Swift structs are a natural home for your web service results - they are immutable, lightweight and have minimal boilerplate code requirements. Getting the results out of JSON and into a struct, however, can be complicated - verifying the correctness of each key in the JSON can quickly lead to an `if let` staircase of doom. 

I wanted to use [something clever and functional](http://robots.thoughtbot.com/efficient-json-in-swift-with-functional-concepts-and-generics), but this has readability drawbacks (I'm not yet comfortable with scattering custom operators all over the place) and also requires me to write a curried initializer for each struct, which seemed like repeating myself a bit too much. 

My design goals were:

- Minimise repetition of code and boilerplate
- Allow for flexibility if the API changes
- Allow for optional properties
- Take advantage of compile-time checking
- Feel at least slightly like I had done something a bit "Swifty"

Here's what I came up with.

<!--more-->

I used [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) for its nice subscripting support and use of optionals. This isn't a post about JSON parsing, that's been done a lot of times. It's a post about the next step.

My web service response structs conformed to a protocol:

{% codeblock lang:swift %}
protocol WebServiceResponse {
    init?(_ json:JSON)
}
{% endcodeblock %}

This is a failable initializer that takes a `JSON` object from SwiftyJSON. An individual response struct looked like this:

{% codeblock lang:swift %}
struct NoticeResponse : WebServiceResponse {
    let noticeID : Int
    let title : String
    let description : String
    let updatedDate : NSDate
    let date : NSDate
    let path : String
    let searchTags : String
}
{% endcodeblock %}

For this particular web service, if anything was missing, then the initializer should fail. Here's the nightmare version of the initialiser:

{% codeblock lang:swift %}
init?(_ json: JSON) {
	if let number = json["id"].int {
		self.noticeID = number
	} else {
		return nil
	}
	if let title = json["title"].string {
		self.title = title
	} else {
		return nil
	}
	// You get the picture...
}
{% endcodeblock %}

Very repetitive, very tedious. Any time code looks like this, you should get a nagging feeling that you could be doing it better. 

I wanted to check for the presence of an optional value, if one existed, assign it to the struct property, otherwise bail out. The curried initializer solution linked above does this, but I really didn't want to have to declare an _n_-step initializer for all of these lightweight result objects.

What I _really_ wanted to do was to be able to create a custom assignment operator so that I could write code like this:

{% codeblock lang:swift %}
self.noticeID =? json["id"].int
{% endcodeblock %}

Where, as a bonus, you'd return `nil` from the whole initialiser if the value didn't exist. This doesn't seem possible, so I ended up with code that looked like this:

{% codeblock lang:swift %}
var m = true
self.noticeID = attempt(json["id"].int, &m)
self.title = attempt(json["title"].string, &m)
...
if !m { return nil }
{% endcodeblock %}

The `m` (for mandatory) flag will be set to false if any of these mandatory values cannot be extracted from the JSON, and will in turn cause the initialiser to fail. Optional properties can be assigned directly from the SwiftyJSON structure.

This meets all of the design goals, and though the `inout` parameter is a bit ugly, hopefully looks understandable to a fresh reader. 

Astute readers will be wondering what the `attempt` function returns if the value isn't present in the JSON. Well, since we'll be binning the struct in that situation anyway, I simply initialise a new value of the appropriate type and return that. To allow that, I had to define the world's simplest protocol:

{% codeblock lang:swift %}
protocol BlankInitable {
    init()
}
{% endcodeblock %}

Then some extensions to cover the types I was using:

{% codeblock lang:swift %}
extension Int : BlankInitable {}
extension String : BlankInitable {}
extension NSDate : BlankInitable {}
{% endcodeblock %}

With that in place, the `attempt` function looked like this: 

{% codeblock lang:swift %}
func attempt<T:BlankInitable>(possible:T?, inout success : Bool) -> T {
    if success {
        if let actual = possible {
            return actual
        } else {
            success = false
        }
    }
    return T()
}
{% endcodeblock %}

Dead simple. If we've already failed, don't bother assessing anything, otherwise attempt to unwrap the optional, return it if present, otherwise fail the flag and return a new blank value.

Since JSON often involves an array of other objects, there's also an array version:

{% codeblock lang:swift %}
func attemptArray<T : WebServiceResponse>(possible: [JSON]?, inout success : Bool) -> [T] {
    if success {
        if let actual = possible {
            var parsedResults = [T]()
            for jsonRepresentation in actual {
                if let parsedResult = T(jsonRepresentation) {
                    parsedResults.append(parsedResult)
                }
            }
            if !parsedResults.isEmpty {
                return parsedResults
            }
        }
    }
    success = false
    return [T]()
}
{% endcodeblock %}

In this case the outer response struct would have an array property of a different `WebServiceResponse` struct. 

I've found this approach very useful, with the additional advantage, thanks to generics, that I only have a single function to call any web service and process the results. In addition to the response structs, I also have a set of configuration structs that hold things like the endpoint, HTTP method and any additional parameters, but that's a topic for a separate post. 