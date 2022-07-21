---
# Please do not edit this file directly; it is auto generated.
# Instead, please edit 025-GETting-data.md in _episodes_rmd/
title: "What is an API?"
teaching: 30
exercises: 15
output: 
  html_document:
    df_print: paged
objectives:
- Understand what an API do
- Connect to Statistics Denmark, and extract data
- Create a list of lists to control the variables to be extracted
keypoints: 
- Getting data from an API is equivalent to requesting a webpage
- POST requests to servers put specific demands on how we request data
source: Rmd
questions:
- What is an API?
editor_options: 
  markdown: 
    wrap: 72
---



Please note: These pages are autogenerated. Some of the API-calls may
fail during that process. We are figuring out what to do about it, but
please excuse us for any red errors on the pages for the time being.

## Using GET

The site icanhazdadjoke.com offers a wide selection of dad-jokes.



a wholesome joke of the type said to be told by fathers with a punchline that is often an obvious or predictable pun or play on words and usually judged to be endearingly corny or unfunny
https://www.merriam-webster.com/dictionary/dad%20joke

In addition to the website, an API is available that can be accessed using the 
GET method.

The GET method is a generic term, we need a function that handles the behind-the-scenes-stuff 
for us. The library *httr* have an implementation:

~~~
library(httr)
~~~
{: .language-r}

Taking a quick look at the documentation we first try GET directly:

~~~
GET("https://icanhazdadjoke.com/")
~~~
{: .language-r}



~~~
Response [https://icanhazdadjoke.com/]
  Date: 2022-07-21 07:47
  Status: 200
  Content-Type: text/html; charset=utf-8
  Size: 9.94 kB
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1, minimum-s...
<meta name="description" content="The largest collection of dad jokes on the ...
<meta name="author" content="C653 Labs" />
<meta name="keywords" content="dad,joke,funny,slack,alexa" />
<meta property="og:site_name" content="icanhazdadjoke" />
...
~~~
{: .output}
What is returned is the response from the server. That includes much more than
what we are looking for. Notable is the "Status" part, which we are told is 
"200", which is server-lingo for "everything is OK".

And what do we get? We get a webpage. We can see that the content is 
*<DOCTYPE html>*. That was not really what we were looking for.

Even if the GET method is relatively simple to work with, we need to add a 
bit more. Again taking a look at the documentation, it appears that we 
need to tell the API, that we would like a specific type of response, rather
than the default html, more specifically "text/plain". 

*httr* has helper functions to assist us. The one we need here is *accept()*


~~~
result <- GET("https://icanhazdadjoke.com/", accept("text/plain")) 
result
~~~
{: .language-r}



~~~
Response [https://icanhazdadjoke.com/]
  Date: 2022-07-21 07:47
  Status: 200
  Content-Type: text/plain
  Size: 82 B
~~~
{: .output}
We still get the response from the server, telling us that Status is 200, and 
everything is OK. But where is our dad-joke?

That is hidden in the content of the response. It is sent to us as binary code,
so we are using the *content()* function, also from *httr* to extract it:


~~~
content(result)
~~~
{: .language-r}



~~~
No encoding supplied: defaulting to UTF-8.
~~~
{: .output}



~~~
[1] "Why did the worker get fired from the orange juice factory? Lack of concentration."
~~~
{: .output}
Theres a little warning about the encoding of the string. But now we have a dad-joke!

What if we need to retrieve a specific joke? All the jokes has an ID, that 
we can use for that. If we want to find that, we need a bit more information about
the joke. We can get that by specifying that we would like the result of our
GET-request returned as JSON. 

shoutout om json.


Looking at the documentation, we see an example, which indicates that what 
we should tell the server that we accept should be "application/json".

~~~
result <- GET("https://icanhazdadjoke.com/", accept("application/json")) 
result
~~~
{: .language-r}



~~~
Response [https://icanhazdadjoke.com/]
  Date: 2022-07-21 07:47
  Status: 200
  Content-Type: application/json
  Size: 125 B
{"id":"pzkbpORS7wc","joke":"Parallel lines have so much in common. It\u2019s ...
~~~
{: .output}
Again - everything is nice and 200 = OK.

We also see a truncated version of the actual joke.

Let us use the content() function to extract the content:


~~~
content(result)
~~~
{: .language-r}



~~~
$id
[1] "pzkbpORS7wc"

$joke
[1] "Parallel lines have so much in common. It’s a shame they’ll never meet."

$status
[1] 200
~~~
{: .output}

status is repeated, and now we have an id. We can use that to extract the 
same joke again.


NOTE: The joke returned is chosen at random. The id used here will probably 
be different from what we found above.

The way to retrieve a specific joke is to GET the URL:

GET https://icanhazdadjoke.com/j/<joke_id>

Where we replace the <joke_id> with the specific joke we want. Remember to
specify the result that we want:


~~~
GET("https://icanhazdadjoke.com/j/lGJmrrzAsc",  accept("text/plain")) %>% 
  content()
~~~
{: .language-r}



~~~
No encoding supplied: defaulting to UTF-8.
~~~
{: .output}



~~~
[1] "A termite walks into a bar and asks “Is the bar tender here?”"
~~~
{: .output}

We can also search for words in jokes. The documentation tells us, that we 
should send our GET request to the URL 
https://icanhazdadjoke.com/search

And in the examples we get the hint, that we should format the URL as:
https://icanhazdadjoke.com/search?term=<TERM>

Dogs are always fun, let us search for dad jokes about dogs. specify the
type of result we want, pipe the response to the content() function and save 
it to *result*:


~~~
result <- GET("https://icanhazdadjoke.com/search?term=dog", accept("application/json")) %>% 
  content()
result
~~~
{: .language-r}



~~~
$current_page
[1] 1

$limit
[1] 20

$next_page
[1] 1

$previous_page
[1] 1

$results
$results[[1]]
$results[[1]]$id
[1] "YvkV8xXnjyd"

$results[[1]]$joke
[1] "Why did the cowboy have a weiner dog? Somebody told him to get a long little doggy."


$results[[2]]
$results[[2]]$id
[1] "82wHlbaapzd"

$results[[2]]$joke
[1] "Me: If humans lose the ability to hear high frequency volumes as they get older, can my 4 week old son hear a dog whistle?\r\n\r\nDoctor: No, humans can never hear that high of a frequency no matter what age they are.\r\n\r\nMe: Trick question... dogs can't whistle."


$results[[3]]
$results[[3]]$id
[1] "GtH6E6UD5Ed"

$results[[3]]$joke
[1] "What kind of dog lives in a particle accelerator? A Fermilabrador Retriever."


$results[[4]]
$results[[4]]$id
[1] "R7UfaahVfFd"

$results[[4]]$joke
[1] "My dog used to chase people on a bike a lot. It got so bad I had to take his bike away."


$results[[5]]
$results[[5]]$id
[1] "obhFBljb2g"

$results[[5]]$joke
[1] "I adopted my dog from a blacksmith. As soon as we got home he made a bolt for the door."


$results[[6]]
$results[[6]]$id
[1] "lyk3EIBQfxc"

$results[[6]]$joke
[1] "I went to the zoo the other day, there was only one dog in it. It was a shitzu."


$results[[7]]
$results[[7]]$id
[1] "71wsPKeF6h"

$results[[7]]$joke
[1] "What did the dog say to the two trees? Bark bark."


$results[[8]]
$results[[8]]$id
[1] "EBQfiyXD5ob"

$results[[8]]$joke
[1] "what do you call a dog that can do magic tricks? a labracadabrador"


$results[[9]]
$results[[9]]$id
[1] "DIeaUDlbUDd"

$results[[9]]$joke
[1] "“My Dog has no nose.” “How does he smell?” “Awful”"


$results[[10]]
$results[[10]]$id
[1] "sPRnOfiyAAd"

$results[[10]]$joke
[1] "At the boxing match, the dad got into the popcorn line and the line for hot dogs, but he wanted to stay out of the punchline."


$results[[11]]
$results[[11]]$id
[1] "AQn3wPKeqrc"

$results[[11]]$joke
[1] "It was raining cats and dogs the other day. I almost stepped in a poodle."


$results[[12]]
$results[[12]]$id
[1] "Lmjqzsr49pb"

$results[[12]]$joke
[1] "What did the Zen Buddist say to the hotdog vendor? Make me one with everything."



$search_term
[1] "dog"

$status
[1] 200

$total_jokes
[1] 12

$total_pages
[1] 1
~~~
{: .output}

This is in JSON format. It is clear that the jokes are in the $results part of that
datastructure. How can we get that to a data frame?

the content() function can treat the content of our response in different ways.
If we treat it as text, the function fromJSON, can convert it to a data frame:

~~~
GET("https://icanhazdadjoke.com/search?term=dog", accept("application/json")) %>% 
  content(as="text") %>% 
  fromJSON()
~~~
{: .language-r}



~~~
Error in fromJSON(.): could not find function "fromJSON"
~~~
{: .error}

We have now seen how to send a request to an API, with search terms 
embedded in the URL.

We have seen how to add an argument to the GET function, that specifies the
type of result we would like.

And we have seen how to extract the results, and get them into a dataframe.

Next, we are going to take a look on how we get results using the POST method,
on an API that provides more factual and serious, but not so funny data.


