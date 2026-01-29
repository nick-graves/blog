+++
date = '2026-01-29T09:26:13-08:00'
title = 'HTB: Magical Palindrome'

image = "/post/images/MagicalPalindrome.jpg"
tags = ["CTF", "HTB", "Web"]
categories = ["HTB"]
+++

# HTB: Magical Palindrome
- Difficulty: Easy
- Category: Web
- Solved: 1/29/2026

## Challenge Description
> In Dumbledore's absence, Harry's memory fades, leaving crucial words lost. Delve into the arcane world, harness the power of JSON, and unveil the hidden spell to restore his recollection. Can you help Harry to find the path to salvation?

![Alt text](/post/images/palindrom_web.jpg)

## Recon
The page has an input form. So the first thing I did was just play around with random inputs. It seemed like no matter what you input (even if it was a valid palindrome) you would get the message "Tootus Shortus". This made sense after looking at the code. The backend was incredibly simple. One endpoint at "/" that took the user's input and called a ```IsPalinDrome``` function. Although, this function is a little different from a traditional palindrome check function (someone needs to brush up on two pointers). The algorithm is as follows:

```javascript
if (string.length < 1000) {
		return 'Tootus Shortus';
	}
```
If the input's length is less than 1000 characters, return "Tootus Shortus".

```javascript
for (const i of Array(string.length).keys())
```
Then loop over a newly created array that is the same length as the input string. It then iterates over this new, blank array. The key here is that we are not actually iterating over the input but rather something that has the same length as the input.

```javascript
const original = string[i];
const reverse = string[string.length - i - 1];
```
We then set some variables as the current `index` and the `final index - the current index` (traditional palindrome logic).

```javascript
if (original !== reverse || typeof original !== 'string') {
		return 'Notter Palindromer!!';
    }
```
Finally we check the two variables that we set (`original` and `reverse`) to see if they are equal OR if the thing at `string[i]` is a string. The return "Notter Palindromer!!" will trigger if EITHER the `original` and `reverse` variables don't match OR `original` is not a string. 

Back to the "/" endpoint, the output of the ```IsPalinDrome``` function is assigned to a variable `error`. If `error` has a value then the endpoint returns status code 400 and `error`'s value. If, however, we can survive all of this logic and keep `error` null we get the following:
```javascript
return c.text(`Hii Harry!!! ${flag}`);
```
So that's what we need to do. We just construct an input that survives all of this logic and the flag will be returned to us.

This seems too easy though. We just need a valid palindrome, that is a string and is longer than 1000 characters. 

Of course this didn't work though. 
![Alt text](/post/images/palindrome_error.jpg)

This is an [Nginx](https://en.wikipedia.org/wiki/Nginx) error message. So our valid solution was never actually reaching the app because Nginx shut it down because it was too large. Looking back at the code, the Nginx config file confirms this:
```
client_max_body_size 75;
```

So Nginx is only allowing requests that are 75 bytes or less. Since each ASCII character is one byte, the maximum possible input would be 75 characters in the *best case*. In practice, the usable input is even smaller because the request body also includes additional bytes for formatting all of which count toward the 75-byte limit.

So that is our problem. We need an input that can 1. pass all of the logic inside the ```IsPalinDrome``` function while also 2. staying under the 75-byte limit of Nginx.


## Exploitation
The app assumes we are going to input a string but *never actually* checks. It just checks the `length` of our input and then compares the values at opposite sides of the input.

So this app really needs 2 things from us:
1. Some length value
2. Matching characters at opposite ends of the input

What if we input some JSON object and just specified the length? If the object we input had some preset value `length`, when the backend calls `string.length` it wouldn't calculate the actual length of the object—it would just reference the value associated with the key `length`.

That gets us through the first checkpoint of the backend logic, but we still need to get through the second if statement. For that, the values of our input at index `i` and `length - 1 - i` need to be the same and they need to be strings. This is where we need to get creative. If in the previous `length` definition, we set it to be a string, when the backend runs ```Array(string.length).keys()``` it evaluates ```Array("1001").keys()``` which produces `0` (since it is only 1 string it will be indexed to 0). So the for loop only runs 1 time. It only compares the `0` index to the `length - 1 - 0` index. When JavaScript tries to evaluate `length - 1 - 0` it will convert length to an int to perform the calculation. Knowing that, we can specify the `0` index and the `length - 1 - 0` index in the JSON input the same way we did for the `length`. All together it looks something like this:

```python 
payload = {"palindrome":{"length":"1001","0":"a","1000":"a"}}
```

On the server's side it will see something like this:
```javascript
{
  length: "1001",
  "0": "a",
  "1000": "a"
}
```

Walking through the backend code, it will first check if the `length` value is less than 1000. Our value `length` would be accessed and converted to an int (thank you, terrible typing in JavaScript) and since 1001 > 1000 we would avoid the return.

Going onto the for loop, `Array(string.length).keys()` would create an empty array of length 1 and get the index which is just 0. 

The `original` and `reverse` variables would be set to `string[i]` and `string[string.length - i - 1]` which in our case would be `string[0]` = "a" and `string[1001 - 0 - 1]` = "a". They are equal and both strings so we would exit out of the `IsPalinDrome` function and get our flag. 

The final script I used can be seen here:
```python
import requests

url = "http://IP:PORT/"
payload = {"palindrome":{"length":"1001","0":"a","1000":"a"}}

response = requests.post(url, json=payload)

print(f"Status: {response.status_code}")
print(f"Response: {response.text}")
```

You could do the same thing through a CURL command:
```bash
curl -X POST http://IP:PORT/ \
-H "Content-Type: application/json" \
-d '{"palindrome": {"length": "1001", "0": "a", "1000": "a"}}'
```

## Takeaways
1. I understand why TypeScript exists. 


