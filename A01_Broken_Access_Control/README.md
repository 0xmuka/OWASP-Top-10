# Broken Access Control

### Agenda:

#### What is Broken Access Control?

#### How Do you find Access Control Vulnerability?

#### How Do you exploit it?

#### How do you prevent it?

## What is Broken Access Control?

> Before learning what **Broken Access Control** is, we should be aware of three concepts:

### 1. **Authentication**:

**Authentication** identifies the user and confirms that **they are who they say they are**.

**Authentication** is the ability to **verify** the user's identity.

**Authentication Example**, if we want to access our personal information or an admin panel, the application will ask us to **log in**.
It checks whether our credentials exist in the database and if they are **valid**.
If everything is correct, we **are successfully authenticated**.

### 2. **Session Management**

**Session Management** identifies **which** subsequent HTTP requests are being made by each user.

**Session Management** **generates a session token** for the user to verify this for every request.

**Session Management Example**: During the authentication process, the user has logged in and verified his credentials.

So, we want to verify his credentials for every request.

In this case, we can **store his identity information** in a session token to verify with this token for every request based on the first login.

In this token, we can **specify** a date to expire the token.

### 3. **Access Control**

**Access Control** determines whether the user is allowed to carry out the action that they are attempting to perform.

**Access Control:** When you perform an action in an application, the session token gets passed to the application.

The backend checks the user ID assigned to this session token.

Once this user is identified, the backend checks the access control roles in the database based on this user ID.

If the user already has the role to perform this action, the request will go through; otherwise, the request will be **access denied**.

> **Broken Access Control** vulnerabilities arise when users act outside of their intended permissions. This typically leads to sensitive information disclosure, unauthorized access and modification or destruction of data.

---

## Three different types of access control:

**Virtical Access Control:** is used to restrict access to functions not available for other users in the organization.

**Horizontal Access Control:** enable different users to access similar resource types.

**Context-Dependent Access Control** restricts access to funtionality and resources based on state of the application or the user's interaction with it.

## Access Control: From Secure Design to Real-World Exploits!

I’ll keep building on the **hospital system example** to highlight the difference between **access control models** (as implemented by developers) and **broken access control types** (as exploited by pentesters when those models are misconfigured).

## **Access Control Models** (Models = Strategy / Design)

These define the **rules** for _who_ should be able to do _what_, _why_, and _under what conditions_.
→ Used by **developers** when designing the system.

### 1. **RBAC – Role-Based Access Control**

- Access is based on **roles**.
- Users are assigned roles like `doctor`, `patient`, `admin`, etc.
- Each role has predefined permissions.

**Hospital example:**

- Doctors can view _all_ patient records.
- Patients can only view their _own_ data.
- Admins can manage users.

**When to use it:**

- When roles are clear and don't change much.
- It simplifies assigning permissions.

### 2. **ABAC – Attribute-Based Access Control**

- Access is based on **attributes** of:

  - the user ( department, location)
  - the resource ( file type, owner)
  - the environment ( time of day, device)

**Hospital example:**

- Only doctors from the **Cardiology department** can access cardiology records.
- Doctors can only access records during working hours ( 9AM–5PM).
- Access is blocked if login is from outside the hospital network.

**When to use it:**

- In complex systems where role alone isn't enough.
- You need _fine-grained control_.

### 3. **DAC – Discretionary Access Control**

- Access is controlled by the **owner** of the resource.
- The owner decides who else can access it.

**Hospital example:**

- A doctor creates a patient case file and decides which other doctors can view it.
- Patients could share their reports with a family member.

**When to use it:**

- In collaborative systems where users can manage their own data sharing (like Google Drive-style sharing).

### 4. **MAC – Mandatory Access Control**

- Access is based on **system-wide security policies** and **clearance levels**.
- Users can’t change permissions — the system strictly controls access.

**Hospital example:**

- Only users with `Confidential Clearance` or higher can access sensitive psychiatric records — even if you're a doctor, you may not have that clearance.
- Access cannot be granted by resource owners.

**When to use it:**

- In highly sensitive environments — military, government, health records.

## **Broken Access Control Types** (Types = Enforcement / Behavior)

These define the **ways access is enforced**, and are **key targets** for **pentesters**.
They test whether the system **correctly applies the rules** at runtime.

### 1. **Vertical Access Control**

- Prevents **low-privileged** users from accessing **high-privileged** functionality.

**Hospital example:**

- A patient shouldn’t be able to access:

  - the admin dashboard
  - doctor-only interfaces
  - medical record editing features

**What pentesters test:**

- Can a patient access admin pages?
- Can a user promote themselves via hidden parameters or direct URLs?

### 2. **Horizontal Access Control**

- Prevents users from accessing **other users’ data** at the same privilege level.

**Hospital example:**

- Patient A must not be able to see Patient B’s reports, even though both are patients.
- A doctor shouldn’t access files of patients not under their care.

**What pentesters test:**

- Try accessing other users’ data by changing IDs or URLs.
- Known as **IDOR** (Insecure Direct Object Reference).

### 3. **Context-Dependent Access Control**

- Access depends on **contextual factors** like:

  - time
  - IP address or location
  - device or browser type
  - previous user interactions or app state

**Hospital example:**

- Doctors can only access the system from within the hospital network.
- Certain functions are only allowed during work hours.
- Admin access is blocked on mobile devices.

**What pentesters test:**

- Try logging in from restricted locations (outside network).
- Attempt to bypass time/device-based restrictions.

### 4. **Access Control Vulnerabilities in Multi-Step Processes**

- These types of vulnerabilities occur when access control rules are implemented on some of the steps, but ignored on others.

**Hospital example:**

- A sensitive operation (patient deletion) requires a confirmation step first — direct access to the delete endpoint is blocked unless it was preceded by the confirmation screen.

**What pentesters test:**

- Directly invoke restricted actions (like POST /delete) without going through the correct flow (like skipping /confirm).
- Modify client-side checks ( using developer tools or intercepting proxies) to force access outside the allowed context.

---

## Other Access Control Examples

- Bypassing access control checks by modifing parameters in the URL or HTML page.
- Accessing the API with missing access controls on the POST, PUT and DELETE methods.
- manipulating metadata, such as replaying or tampering with JSON Web Tokens (JWTs) or a cookie.
- Exploiting CORS misconfiguration that allow API access from unauthorized / untrusted origins.
- Force browsing to aunthenticated pages as an aunthenticated user.

## Spot the Vulnerability?
```
1 public boolean deleteOrder(Long id){
2       Order order = orderRepository.getOne(id);
3       if (order == null) {
4           log.info("No found order");
5           return false;
6       }
7       User user = order.getUser();
8       orderRepository.delete(order);
9       log.info("Delete order for user {}", user.getId());
10      return true;
11 }
```
<details>
<summary>Answer</summary>

* The API endpoint deletes orders by ID, but does not verify if this order has been made by the current logged-in user.
* This presents an opportunity for an attacker to exploit this loophole and delete the orders of other users.
</details>


$-$For safe access restrictions to be properly implemented, the code would look more like this:

<details>
<summary>Answer</summary>

```
1 public boolean deleteOrder(Long id) {
2        User user = userService.getUserByContext();
3        boolean orderExist = getUserOrders().stream()
4                .anyMatch(order -> (order.getId() == id));
5        if (orderExist) {
6            orderRepository.deleteById(id);
7            log.info("Delete order for user {}", user.getId());
8            return true;
9        } else {
10            log.info("No found order");
11            return false;
12        }
13 }
```
</details>

## Impact of Access Control Vulnerabilites

### Unauthorized access to application.
* **C**onfidentiality $-$ Access to other user's data.
* **I**ntegrity $-$ Access to update other user's data.
* **A**vailability $-$ Access to delete users.

> #### Can some times be chained with other vulnerabilities to gain remote code excution (RCE) on the host operating system.
## How to find Access Control Vulnerabilities?

**White Box Testing:** The tester has full internal knowledge (code, architecture, access) to perform a deep, targeted analysis of vulnerabilities.

**Black Box Testing:** The tester has zero prior knowledge, simulating an external attacker to assess real-world breach potential.

**Gray Box Testing:** The tester has partial knowledge (user-level access or system overview), simulating an insider or partner with limited access to identify potential internal threats.

> **Note:**
>* Pure Black Box testing is too limited for our needs, so we’ll use a **Gray Box approach** — a mix of internal and external knowledge.
>* When we mention "Black Box" here, we actually mean **Gray Box** for a more realistic test.

## Black $-$ box Testing

* **Map the Application** by running Burp Suite or ZAP in the background silently and thoroughly evaluating all functionalities of the application.

* Identify instances where the web application interacts with the underlying operating system, revealing potential vulnerabilities.
* Analyze how access control is implemented at each privilege level within the application.
* Manipulate parameters that may influence access control decisions on the backend to test security mechanisms.
* Automate testing processes using tools like the Autorize extension for efficient vulnerability discovery.
 
## White $-$ box Testing

* Review the code to identify how access control is implemnted in the application.
  - System defaults to open.
  - weak or missing access control checks on functions / resources.
  - Missing access control rules for POST, PUT and DELETE methods at the API level.
  - Relying solely on client$-$side input to perform access control decisions.
  - Validate potential access control vulnerabilities on a running application.
