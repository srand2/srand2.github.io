
# IDOR

![[Pasted image 20210901195610.png]]![Event Handlers](/docs/assets/images/20210901195610.png)

Insecure Direct Object Reference refers to an object that is exposed to anyone who requests it **directly**

![[Pasted image 20210901200022.png]]![Event Handlers](/docs/assets/images/20210901200022.png)

![[Pasted image 20211125110655.png]]![Event Handlers](/docs/assets/images/20211125110655.png)

![Event Handlers](/docs/assets/images/Pasted%20image%2020211125110731.png)

In our first example, whats stopping someone from checking another users ID and getting all their information?

![Event Handlers](/docs/assets/images/Pasted%20image%2020210901200131.png)

This code example demonstrates how this can occur.

![Event Handlers](/docs/assets/images/Pasted%20image%2020210901200314.png)

The server is taking in the users id and directly displaying that information.

To fix this the server shouldnt believe the user, but rather extract the session ID and check on the backend.

![Event Handlers](/docs/assets/images/Pasted%20image%2020210901200459.png)

In case of Serverless checks, the application must have a mechanism to check a signature (JWT)

![Event Handlers](/docs/assets/images/Pasted%20image%2020210901200557.png)

## Forced Browsing

Similar to IDOR, forced browsing can occur if the application exposes a direct reference to a file location

![Event Handlers](/docs/assets/images/Pasted%20image%2020210901200849.png)

Here an attacker can visit the link and retrieve the image even though it is a private image


## UUID 

![Event Handlers](/docs/assets/images/Pasted%20image%2020210901201023.png)

Consider the server accepts arbitrary uuid's from the user, yet the uuid is too long and random to guess. What can an attacker perform?

A GUID can foil your plans and make it really hard to exploit this hypothetical IDOR scenario. So when you come across one try to create a few users in order to get their GUID so you can try to understand how the GUID is formed. If there is no specific pattern then only thing to do is to enumerate the GUID. Sometimes there faults in the API that can leak the GUID, sometimes CORS misconfiguration can like it too, so you need to try and find a way to enumerate the GUID and perform IDOR.

One option is to try and find a location the uuid gets leaked.

if you face an encoded value, you can test the idor vulnerability with decoding the encoded value. if you face a hashed value, you should test whether the hash value is an accessible or predictable value.

![Event Handlers](/docs/assets/images/Pasted%20image%2020211207105115.png)

Looking at the user id of “8f14e45fceea167a5a36dedd4bea2543” you might think it's a random id that's impossible to guess but that may not be the case. It's common practice to hash user ids before storing them in a database so maybe that's what's happening here.

![Event Handlers](/docs/assets/images/Pasted%20image%2020211207105204.png)

As you can see above this is a MD5 hash of the number 7. If an attacker were to take an MD5 Hash of the number “11” they would be able to craft a user id for that user.

![Event Handlers](/docs/assets/images/Pasted%20image%2020211207105219.png)

Now that we generated an MD5 hash for the integer 11 we can use this to retrieve information from that person's user account.

![Event Handlers](/docs/assets/images/Pasted%20image%2020211207105239.png)

(bugbountyplaybook2)

![Event Handlers](/docs/assets/images/Pasted%20image%2020220117111509.png)


## Bypassing Validation 

https://blog.mert.ninja/idor/


## How To Think

1.  **Understand the business logic of the application  
    **Don’t be on auto pilot and don’t change random IDs in API calls until you see PII in the response. There are automatic tools that can do this for you. Think.
2.  **Every time you see a new API endpoint that receives an object ID from the client, ask yourself the following questions:**  
    * Does the ID belong to a private resource? For example, If we talk about some “news” feature, and the API endpoint is “_/api/articles/555_”, there’s a good chance that all the objects should be public by design, since articles are public.  
    * What are the IDs that belong to me?
3.  **Understand relationships between resources  
    **Understand that there’s a relationship between “trips” and “receipts” and that each trip belongs to a specific user.
4.  **Understand the roles and groups in the API  
    **Try to understand what the different possible roles in the API are. For example — user, driver, supervisor, manager.
5.  **Leverage the predictable nature of REST APIs to find more endpoints**  
    You saw some endpoint that exposes a resource in a RESTy way?  
    For example: _GET /api/chats/<chat_id>/message/<message_id>_  
    Don’t be shy, try to replace GET with another [HTTP method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods).  
    If you receive an error, try to:  
    * Add a “Content-length” HTTP header  
    * Change the “Content-type”


## How To Test:

The basic way to test for BOLA is to guess the random ID of the object, but this doesn’t always work. The more efficient way would be to perform “Session Label Swapping”:

1.  Identify the session label:  
    A session label is a generic term to describe every string that is used by the API to identify the logged in user. The reason I call it a session label and not a session ID or authentication token is because, for BOLA exploitation, it doesn’t matter!
2.  Log two session labels from different users:  
    Create two different users and document their session label.  
    For example:  
      
    _User 1, Hugo =  
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMTEiLCJuYW1lIjoiSHVnbyIsImlhdCI6MTUxNjIzOTAyMn0.JaTvHH5TbKzgRMa5reRDMiPjHEmPKY8axsu5dFMu5Ao  
    _  
    _User 2, Bugo =  
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIyMjIiLCJuYW1lIjoiQnVnbyIsImlhdCI6MTUxNjIzOTAyMn0.uXTOzhX1mnYI3TZUSyjWS7qQXfYNn6qw-MehVGAgvpc  
    _  
    Keep them in a .txt file.
3.  Login as user #1, sniff the traffic and find an API call to an endpoint that receives an object ID from the client. For example: _/api/trips/_**_e6d84adc-b1c7–4adf-a392–35e5b71f068a_**_/details_
4.  Repeat and intercept the suspicious API call.
5.  Change the session label of user #1 to the session label of user #2 — in order to make a call on behalf of user #2x.
6.  Absorb the result. If you received an authorization error, the endpoint is probably not vulnerable.
7.  If the endpoint returns details of the object, compare the two results and check if they are the same. If they are the same — the endpoint is vulnerable.

-   Some endpoints don’t return data. For example, the endpoint “_DELETE /api/users/<user_id>_” would only return a status code without data.  
    It might be a bit more complicated to understand if they are vulnerable or not, but don’t ignore them.

## _Tips & Tricks:_

-   Object IDs in URLs tend to be less vulnerable. Try to put more effort on IDs in HTTP headers / bodies.
-   GUID instead of numeric values? Don’t give up!  
    Use the “session label swapping” technique or find endpoints that return IDs of objects that belong to other users.
-   Always try numeric IDs. If you found that an endpoint receives a non-numeric object ID, like a GUID or an email address, give it a shot and try to replace it with a numeric value (e.g.: replace “[inon@](mailto:inon@email.com)traceable.ai“ with the number “100”)
-   Received 403/401 once? Don’t give up!  
    It’s not extremely common, but some weird authorization mechanisms only work partially. Try many different IDs. For example: if the endpoint “_api/v1/trips/666_” returned 403, run a script to enumerate 50 random IDs from 0001 to 9999.
-   Find the most niche features in the application  
    Let’s say you found a weird feature to create a customized picture to your profile only during avocado awareness month (it’s June btw), and it performs an API call to _/api/avocado_awarness_month/profile_pics/<avocado_emoji_id>_.  
    There’s a very good chance that the developers haven’t thought about authorization there.

## How to bypass object level authorization:

-   Wrap the ID with an array.  
    Instead of {“id”:111} send {“id”:[111]}
-   Wrap the ID with a JSON object  
    Instead of {“id”:111} send {“id”:{“id”:111}}
-   Try to perform [HTTP parameter pollution](https://medium.com/@0xgaurang/case-study-bypassing-idor-via-parameter-pollution-78f7b3f9f59d):  
    _/api/get_profile?user_id=<legit_id>&user_id=<victim’s_id>_  
    OR  
    _/api/get_profile?user_id=<victim’s_id>&user_id=<user_id>_  
    This can work in situations where the component that performs the authorization check and the endpoint itself use different libraries to parse query parameters. In some cases, library #1 would take the first occurence of “user_id” while library #2 would take the second.
-   Try to perform [JSON parameter pollution  
    ](http://blog.blueinfy.com/2018/07/json-parameter-pollution.html)_POST api/get_profile  
    {“user_id”:<legit_id>,”user_id”:<victim’s_id>}_  
    OR  
    _POST api/get_profile  
    {“user_id”:<victim’s_id>,”user_id”:<legit_id>}_  
    It’s pretty similar to HTTP parameter pollution.
-   Try to send a wildcard instead of an ID. It’s rare, but sometimes it works.
-   Find API hosts where the authorization mechanism isn’t enabled. Sometimes, in different environments, the authorization mechanism might be disabled, for various reasons. For example: QA would be vulnerable but production would not.
-   Try to perform [type-juggling](https://blog.sucuri.net/2017/02/content-injection-vulnerability-wordpress-rest-api.html)
-   Not common, but a great example of how you can [bypass BOLA protection](https://web.archive.org/web/20190614232925/https://medium.com/intigriti/how-spending-our-saturday-hacking-earned-us-20k-60990c4678d4)


## How to Test - In Pictures

![Event Handlers](/docs/assets/images/Pasted%20image%2020211125111427.png)

![Event Handlers](/docs/assets/images/Pasted%20image%2020211125111438.png)

![Event Handlers](/docs/assets/images/Pasted%20image%2020211125111524.png)


## Autorize 
### **How to use it?**

First, to avoid being polluted by [resources](https://blog.yeswehack.com/resources-vulnerability-disclosure/ "resources") or URLs that are not inside your scope, we recommend to start by adding a new Interception Filters. Go on _**Interception Filters > Select “Scope items only: (Content is not required)**_ and click on **Add filter** (be sure to have a scope set in _**Target > Scope**_).

![](https://blog.yeswehack.com/wp-content/uploads/image-8.png.webp)

Once this is done, open your web browser and create two accounts on your target scope (**[attacker]** and **[victim]**). Log on to your **[attacker]** account and in the _**Intercept**_ or _**proxy**_ tab, copy the cookie of your **[attacker]** session (this may also be a header string like JWT token) and paste the value to the configuration inside the Autorize tab.

![](https://blog.yeswehack.com/wp-content/uploads/image-5.png.webp)

You’re now ready to test for IDORs from your **[victim]** account. On Autorize, click on **_Authorize is off_** to start. Autorize can do many things for you, but you also need to work for him 🙂

From your current **[victim]** account session, try as many things you can on your target: add new data, tamper with your data, delete them, test all the features, the goal here is to accumulate the maximum number of queries inside the Autorize tab.

![](https://blog.yeswehack.com/wp-content/uploads/image-9.jpg.webp)

#### What’s going on? It’s like a Christmas tree

To clearly identify potentials IDORs, Autorize use colors highlighting in **Authz Status** and **Unauth Status** columns:

-   **Red “bypassed!”**: endpoint _could_ be vulnerable to IDOR,
-   **Orange “Is enforced!”**: endpoint _seems_ to be protected but look anyway,
-   **Green “Enforced!”**: endpoint is _clearly_ protected against IDOR.

Be careful, Autorize displaying many red highlight requests does not mean that all endpoints are systematically vulnerable. There may be false positives, it’s up to you to double-check the output.

Now compare the responses size between each query: if it’s the same, go deeper!:

-   **Orig. Len**: is the _size of the response_ from our original session (our **[victim]** account)
-   **Modif. Len**: is the response from the _same request_ replayed by Autorize using the **[attacker]** cookies (_automatically replayed by Autorize_)
-   **Unauth. Len**: is also the same request as our victim account but without cookies session (to test if the endpoint is vulnerable to _Improper Access Control_)

![](https://blog.yeswehack.com/wp-content/uploads/image-6.png.webp)

Click on the interesting request and check the **_Request/Response Viewers_** tab (right):

![](https://blog.yeswehack.com/wp-content/uploads/image-7.png.webp)

_**Modified Response**_ correspond to the server response to our **[victim**] request but with our **[attacker]** cookies. As we can see, many information are displayed and a vulnerability of the IDOR type seems to exist.

**_Original Response_** correspond to the server response to our original **[victim]** request. It’s useful to compare the response between our two accounts and to detect false positive.

**_Unauthenticated Response_** is the server response to the request but without any auth cookie.

By using this plugin, you should have a better chance of finding _IDOR_ and _Improper Access Control_ vulnerabilities. Moreover, it allows you to automate some queries and focus on the interesting ones.
![Event Handlers](/docs/assets/images/Pasted%20image%2020211125111653.png)


## Resources
[BOLA](https://inonst.medium.com/a-deep-dive-on-the-most-critical-api-vulnerability-bola-1342224ec3f2)

