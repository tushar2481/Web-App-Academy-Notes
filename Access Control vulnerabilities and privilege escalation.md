# Access Control vulnerabilities and privilege escalation

## What is Access Control?

Access control (or authorization) is the application of constraints on who (or what) can perform attempted actions or access resources that they have requested. In the context of web applications, access control is dependent on authentication and session management:

- **Authentication** identifies the user and confirms that they are who they say they are.
- **Session management** identifies which subsequent HTTP requests are being made by that same user.
- **Access control** determines whether the user is allowed to carry out the action that they are attempting to perform.

Access control design decisions have to be made by humans, not technology, and the potential for errors is high. mainly divided in 3 category.

## **What are access control security models?**

An access control security model is a formally defined definition of a set of access control rules that is independent of technology or implementation platform. implemented within operating systems, networks, database management systems and back office, application and web server software.

**Programmatic access control** matrix of priviledge stored DB and used by programming

**Discretionary access control (DAC)** Owners of resources or functions have the ability to assign or delegate access permissions to users

**Mandatory access control (MAC)** a centrally controlled system of access control in which access to some object (a file or other resource) by a subject is constrained.

**Role-based access control (RBAC),** named roles are defined to which access privileges are assigned. Users are then assigned to single or multiple roles.

### Vertical access controls

Vertical access controls are mechanisms that restrict access to sensitive functionality that is not available to other types of users. With vertical access controls, different types of users have access to different application functions.

### Horizontal access controls

Horizontal access controls are mechanisms that restrict access to resources to the users who are specifically allowed to access those resources. With horizontal access controls, different users have access to a subset of resources of the same type.

### Context-dependent access controls

Context-dependent access controls restrict access to functionality and resources based upon the state of the application or the user's interaction with it. Context-dependent access controls prevent a user performing actions in the wrong order.

## Vertical privilege escalation :

If a user can gain access to functionality that they are not permitted to access then this is vertical privilege escalation. For example, if a non-administrative user can in fact gain access to an admin page where they can delete user accounts, then this is vertical privilege escalation.

**Unprotected Functionality :**

arises where an application does not enforce any protection over sensitive functionality. For example, administrative functions might be linked from an administrator's welcome page but not from a user's welcome page. However, a user might simply be able to access the administrative functions by browsing directly to the relevant admin URL. suppose [https://insecure-web.com/admin](https://insecure-web.com/admin) this can be access by any user or it might be disclosed at any location like [https://insecure-web.com/robots.txt](https://insecure-web.com/robots.txt). bruteforcing is most basic to find this even if not disclosed anywhere.

- Lab-1 Solution:
    
    Go to the lab and view robots.txt by appending /robots.txt to the lab URL. Notice that the Disallow line discloses the path to the admin panel. In the URL bar, replace /robots.txt with /administrator-panel to load the admin panel. Delete carlos.
    

In some cases, sensitive functionality is not robustly protected but is concealed by giving it a less predictable URL: [https://insecure-web.com/administrator-panel-y540/](https://insecure-web.com/administrator-panel-y540/) still it might be leaked by application though not predictable. 

- Lab-2 Solution:
    
    Review the lab home page's source using Burp Suite or your web browser's developer tools. Observe that it contains some JavaScript that discloses the URL of the admin panel. Load the admin panel and delete carlos.
    

**Parameter-based access control methods:**

Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location, such as a hidden field, cookie, or preset query string parameter & make access control decision based on submitted values. eg : [https://insecure-web.com/home.jsp?admin=true](https://insecure-web.com/home.jsp?admin=true) , [https://insecure-web.com/home.jsp?role=1](https://insecure-web.com/home.jsp?role=1) , [https://insecure-web.com/home.jsp/f9bbf94dcd2120570d411ac47ae752e3/](https://insecure-web.com/home.jsp/f9bbf94dcd2120570d411ac47ae752e3/) (here md5 hash is role=admin)

- Lab-1 Solution :
    
    by login through my account, i noticed header admin=false, so sent request in repeater and changed header admin=true. so i've seen admin panel. now in burp's option tab, i used match & replace functionality and set admin=false to admin=true. now again try for admin panel. now i can delete user carlos and lab get solved.
    
- Lab-2 Solution :
    
    login through user credential, now there is update mail functionality, send request with new mail id and submit it. now monitor through burp history and notice that change mail's response contain roleid:1 so send request related response to repeater and add "roleid":2 in JSON body format and we can see in repeaters response id is changed. we got admin panel. now we can delete user from there.
    

**Broken access control resulting from platform misconfiguration:**

Some applications enforce access controls at the platform layer by restricting access to specific URLs and HTTP methods based on the user's role. For example an application might configure rules like the following:

`DENY: POST, /admin/deleteUser, managers`

it will deny access to given URL with POST method to group managers.

Some application frameworks support various non-standard HTTP headers that can be used to override the URL in the original request, such as `X-Original-URL` and `X-Rewrite-URL`.

- Lab-1 Solution :
    
    we can directly see admin panel but when we click it we get blocked. and it's simple message "Access Denied" now monitor this request in burp and use X-Original-URL: /admin header and remove admin from 1st header. we got access to admin panel. so, application frameworks support various non-standard HTTP headers. use X-Original-URL: /admin/delete and username=carlos in body and GET / HTTP/1.1 and send request we have deleted user.
    

An alternative attack can arise in relation to the HTTP method used in the request. restriction may also occur over HTTP method. 

- Lab-2 Solution : (solution only applicable when u have admin panel access) method of changing request's HTTP method can applicable to another points also
    
    in main tab login with admin credentials and upgrade carlos user. now in privet window login with your username wiener and copy wiener's cookie. now monitor upgradation request by admin in history and send it to repeater, now just change cookie and send request - "unauthorized"  now change method POST to POSTX - "method not allowed" so change it to get by right click on request and select change request method and send request. it has done successfully, now change username=wiener and send. now u r also admin.
    

---

## Horizontal privilege escalation:

**Simple Horizontal Privilege escalation:**

Horizontal privilege escalation arises when a user is able to gain access to resources belonging to another user, instead of their own resources of that type. For example, if an employee should be able to access only own employment and payroll records, but can in fact also access the records of other employees.

`https://insecure-website.com/myaccount?id=123`

Now, if an attacker modifies the `id` parameter value to that of another user, then the attacker might gain access to another user's account page, with associated data and functions.

- Lab-1 Solution :
    
    login using our creds. go to "my account" functionality. we can see username in id parameter. send request to repeater and change username and get API key
    

In some applications, the exploitable parameter does not have a predictable value. For example, instead of an incrementing number, an application might use globally unique identifiers (GUIDs) to identify users. Here, an attacker might be unable to guess or predict the identifier for another user. However, the GUIDs belonging to other users might be disclosed elsewhere in the application where users are referenced, such as user messages or reviews.

- Lab-2 Solution :
    1. Find a blog post by `carlos`.
    2. Click on `carlos` and observe that the URL contains his user ID. Make a note of this ID.
    3. Log in using the supplied credentials and access your account page.
    4. Change the "id" parameter to the saved user ID.
    5. Retrieve and submit the API key.

when the user is not permitted to access the resource, and returns a redirect to the login page. However, the response containing the redirect might still include some sensitive data belonging to the targeted user, so the attack is still successful.

- Lab-3 Solution :
    1. Log in using the supplied credentials and access your account page.
    2. Send the request to Burp Repeater.
    3. Change the "id" parameter to `carlos`.
    4. Observe that although the response is now redirecting you to the home page, it has a body containing the API key belonging to `carlos`.
    5. Submit the API key.

**Horizontal to Vertical privilege escalation :**

Often, a horizontal privilege escalation attack can be turned into a vertical privilege escalation, by compromising a more privileged user. For example, a horizontal escalation might allow an attacker to reset or capture the password belonging to another user.

- Lab Solution :
    1. Log in using the supplied credentials and access the user account page.
    2. Change the "id" parameter in the URL to "administrator".
    3. View the response in Burp and observe that it contains the administrator's password.
    4. Log in to the administrator account and delete `carlos`.

**Insecure direct object references (IDOR) :**

Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly.

*IDOR vulnerability with direct reference to database objects :*

`https://insecure-website.com/customer_account?customer_number=132355`

Here, the customer number is used directly as a record index in queries that are performed on the back-end database. If no other controls are in place, an attacker can simply modify the customer_number value, bypassing access controls to view the records of other customers. This is an example of an IDOR vulnerability leading to horizontal privilege escalation.

*IDOR vulnerability with direct reference to static files :*

IDOR vulnerabilities often arise when sensitive resources are located in static files on the server-side filesystem. For example, a website might save chat message transcripts to disk using an incrementing filename, and allow users to retrieve these by visiting a URL like the following:

`https://insecure-website.com/static/12144.txt`

- Lab Solution :
    1. Select the "Live chat" tab.
    2. Send a message and then select "View transcript".
    3. Review the URL and observe that the transcripts are text files assigned a filename containing an incrementing number.
    4. Change the filename to `1.txt` and review the text. Notice a password within the chat transcript.
    5. Return to the main lab page and log in using the stolen credentials.

**Access control vulnerabilities in multi-step processes :**

This is often done when a variety of inputs or options need to be captured, or when the user needs to review and confirm details before the action is performed. For example, administrative function to update user details might involve the following steps:

1. Load form containing details for a specific user.
2. Submit changes.
3. Review the changes and confirm.

Sometimes, a website will implement rigorous access controls over some of these steps, but ignore others. For example, suppose access controls are correctly applied to the first and second steps, but not to the third step. Effectively, the website assumes that a user will only reach step 3 if they have already completed the first steps, which are properly controlled. Here, an attacker can gain unauthorized access to the function by skipping the first two steps and directly submitting the request for the third step with the required parameters.

- Lab Solution : (Pending)

**Referer-based access control :**

Some websites base access controls on the `Referer` header submitted in the HTTP request. The `Referer` header is generally added to requests by browsers to indicate the page from which a request was initiated.

For example, suppose an application robustly enforces access control over the main administrative page at `/admin`, but for sub-pages such as `/admin/deleteUser` only inspects the `Referer` header. If the `Referer` header contains the main `/admin` URL, then the request is allowed.

In this situation, since the `Referer` header can be fully controlled by an attacker, they can forge direct requests to sensitive sub-pages, supplying the required `Referer` header, and so gain unauthorized access.

- Lab Solution : (Pending)

**Location-based access control :**

Some web sites enforce access controls over resources based on the user's geographical location. This can apply, for example, to banking applications or media services where state legislation or business restrictions apply. These access controls can often be circumvented by the use of web proxies, VPNs, or manipulation of client-side geolocation mechanisms.

---

## How to prevent access control vulnerabilities :

Access control vulnerabilities can generally be prevented by taking a defense-in-depth approach and applying the following principles:

- Never rely on obfuscation alone for access control.
- Unless a resource is intended to be publicly accessible, deny access by default.
- Wherever possible, use a single application-wide mechanism for enforcing access controls.
- At the code level, make it mandatory for developers to declare the access that is allowed for each resource, and deny access by default.
- Thoroughly audit and test access controls to ensure they are working as designed.