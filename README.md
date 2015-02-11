# MBWebServicesExample

### Web Services Defined
Many companies like Facebook and Twitter make their services available to users via web sites. 
Often these services are also available to developers to use in their apps; these are called web services. 
Web services are functions and content that reside on a web server that you can use via a well-defined set of rules called an API (Application Programming Interface).

The general pattern to working with web services is to formulate a request, send the request, receive the response, and then interpret the response. 
Objective-C comes with support for web services. 
To send requests and receive responses you can use the **NSURLConnection** class with the **NSData** class. 
To interpret, or parse, the response, you can use the **NSJSONSerialization** class.

JSON stands for JavaScript Object Notation and is used for data storage and transportation. 
Web services that are implemented as REST (Representational State Transfer) web services will provide JSON response data. 
The **NSJSONSerialization** class makes working with JSON easier in Objective-C.

### Bitly Example
Bitly is a good example of a web service that I like to use as a demonstration because it is pretty simple and provides a very clear function. 
Bitly will take a long URL (the string that you type into a web browser) and turn it into a short URL that is easier to type. 
I am going to use the bitly web service to showcase the **NSURLConnection** class.

*Note: To follow along with this recipe, you will need to create a free account with bit.ly and get your own API key and API username. 
Go to https://bit.ly to get your account if you wish to follow along with this example.  
In the examples, when I include [YOUR API LOGIN] or [YOUR API KEY] you will need to substitute the login and key that you obtained from bitly.*

### Formulate Request String
When you work with a web service, you should use the documentation provided by the company that published the web service as a reference. 
This documentation will give you a string and parameters that you can use. 
You are going to use the string from the API documentation(http://api.bit.ly/shorten?version=2.0.1&longUrl=&login=&apiKey=&format=json) 
as a starting point along with the bitly login, bitly key, and a long URL as parameters to formulate your request string.

    NSString *APILogin = @"[YOUR API LOGIN]";
    NSString *APIKey = @"[YOUR API KEY]";
    NSString *longURL = @"https://mobileappmastery.com";
    NSString *requestString = [NSString stringWithFormat:@"http://api.bit.ly/shorten?version=2.0.1&longUrl=%@&login=%@&apiKey=%@&format=json", longURL, APILogin, APIKey];

### Create the Session and URL
You are going to need two objects: 
an **NSURL** object to represent the request URL that you are sending to the server and a **NSURLSession** object to do your web work for you.

    NSURL *requestURL = [NSURL URLWithString:requestString];
    NSURLSession *session = [NSURLSession sharedSession];

### Send and Receive the Response
You are going to use a block with the **NSURLSession** object to ask the web service to shorten the URL. 
Put all the code that you need to work with the response in the block that you send as a parameter. 
You are really doing two things at once with this method.

    [[session dataTaskWithURL:requestURL
      completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        // do stuff in block
     }] resume];
 
    sleep(60);

You still need to fill in the block where you handle the response, but this will start the action. 
Notice that you need to put a **sleep** function in this code. 
The **sleep** function will stop new code from executing on the main thread for 60 seconds. 
You need this because the method you are using is going to execute on a background thread (this is the best practice when using web services). 
If you don’t stop the command line app from finishing the web service, it won’t have enough time to fetch the results for you.

### Parsing JSON
Inside the completion block, you can add the code used to interpret the response from the web server. 
Since you know that you are going to be working with JSON, you will use the **NSJSONSerialization** class. 
You need an **NSError** object here as well as the **NSData** object supplied by the block (this contains the data from the web service response).

    [[session dataTaskWithURL:requestURL
      completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
         NSError *e = nil;
         NSDictionary *bitlyJSON = [NSJSONSerialization JSONObjectWithData:data options:0 error:&e];
     }] resume];

This gives you all the JSON data organized in an **NSDictionary** collection. 
This dictionary can have other dictionaries, arrays, numbers, and strings located inside it. 
The next step is a process of going through all these returned objects to locate what you need.  
You also need to test for errors here.

    [[session dataTaskWithURL:requestURL
      completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
          
         NSError *e = nil;
         NSDictionary *bitlyJSON = [NSJSONSerialization JSONObjectWithData:data options:0 error:&e];
         
         if(!error) {
           NSDictionary *results = [bitlyJSON objectForKey:@"results"];
           NSDictionary *resultsForLongURL = [results objectForKey:longURL];
           NSString *shortURL = [resultsForLongURL objectForKey:@"shortUrl"];
           NSLog(@"shortURL = %@", shortURL);
         } else
             NSLog(@"There was an error parsing the JSON");
     }] resume];

Once this is all set up, if you run your app, 
you will have retrieved the **shortURL** from the response and printed the following out to your console log:

    shortURL = http://bit.ly/1fHrAsT
  
*Note: When you are parsing a web service response like this, 
you will need to investigate where the important data is by looking at the API documentation or viewing the string that is returned.*
