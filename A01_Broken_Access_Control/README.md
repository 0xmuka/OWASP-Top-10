# Broken Access Control

### Table of Contents

- #### [What is Broken Access Control?](#what-is-broken-access-control)

- #### [Different Types of Access Control?](#three-different-types-of-access-control)

- #### [Access Control Models](#access-control-models-models--strategy--design)

- #### [Broken Access Control Types](#broken-access-control-types-types--enforcement--behavior)

- #### [How Do you find Access Control Vulnerability?](#how-to-find-access-control-vulnerabilities)

- #### [How Do you exploit it?](#how-to-exploit-access-control-vulnerabilities)

- #### [How do you prevent it?](#how-to-prevent-access-control-vulnerabilites)

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

- The API endpoint deletes orders by ID, but does not verify if this order has been made by the current logged-in user.
- This presents an opportunity for an attacker to exploit this loophole and delete the orders of other users.
</details>

$-$ For safe access restrictions to be properly implemented, the code would look more like this:

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

- **C**onfidentiality $-$ Access to other user's data.
- **I**ntegrity $-$ Access to update other user's data.
- **A**vailability $-$ Access to delete users.

> #### Can some times be chained with other vulnerabilities to gain remote code excution (RCE) on the host operating system.

## How to find Access Control Vulnerabilities?

**White Box Testing:** The tester has full internal knowledge (code, architecture, access) to perform a deep, targeted analysis of vulnerabilities.

**Black Box Testing:** The tester has zero prior knowledge, simulating an external attacker to assess real-world breach potential.

**Gray Box Testing:** The tester has partial knowledge (user-level access or system overview), simulating an insider or partner with limited access to identify potential internal threats.

> **Note:**
>
> - Pure Black Box testing is too limited for our needs, so we’ll use a **Gray Box approach** — a mix of internal and external knowledge.
> - When we mention "Black Box" here, we actually mean **Gray Box** for a more realistic test.

## Black $-$ box Testing

- **Map the Application** by running Burp Suite or ZAP in the background silently and thoroughly evaluating all functionalities of the application.

- Identify instances where the web application interacts with the underlying operating system, revealing potential vulnerabilities.
- Analyze how access control is implemented at each privilege level within the application.
- Manipulate parameters that may influence access control decisions on the backend to test security mechanisms.
- Automate testing processes using tools like the Autorize extension for efficient vulnerability discovery.

## White $-$ box Testing

- Review the code to identify how access control is implemnted in the application.
  - System defaults to open.
  - weak or missing access control checks on functions / resources.
  - Missing access control rules for POST, PUT and DELETE methods at the API level.
  - Relying solely on client $-$ side input to perform access control decisions.
  - Validate potential access control vulnerabilities on a running application.

## How to exploit Access Control vulnerabilities?

- Depends on the type of access control vulnerability.
- Usually just a matter of manipulated the vulnerable field / parameter.

- > **Note:** I will walk through different types of access control vulnerabilities with PortSwigger's 13 labs, explaining each lab in a separate README file.

### Automated Exploitation Tools

1. **Arachni** - Open-source web application security scanner framework
2. **Wapiti** - Open-source web vulnerability scanner
3. **Acunetix** - Commercial web vulnerability scanner
4. **w3af** (Web Application Attack and Audit Framework) - Open-source web application security scanner
5. **OWASP ZAP** (Zed Attack Proxy) - Open-source web application security scanner and penetration testing tool
6. **Burp Suite** (Community/Professional) - Comprehensive web security testing platform with scanner and proxy

### Extentions

#### **Burp Suite** [Burp Extention $-$ Autorize](https://portswigger.net/bappstore/f9bbac8c4acf4aefa4d7dc92a991af2f)

- > Autorize is a Burp Suite extension designed to help penetration testers automatically detect authorization vulnerabilities, which are often time-consuming to test manually during a web application security assessment.

<details>
<summary>Summary</summary>

## 🧠 **How Does It Work?**

The extension works by **repeating requests** with different sessions and checking whether **access control is properly enforced**.

### Main idea:

1. You log in as a **high-privileged user** (admin, doctor).
2. You provide Autorize with the **cookies of a low-privileged user** (a normal user or patient).
3. As you browse the application, Autorize **automatically re-sends each request**:

   - Using the **low-privileged user's cookies**
   - And optionally **with no cookies at all** (to test unauthenticated access)

Then it compares the responses to detect if there is a **Broken Access Control** vulnerability.

---

## 🛡️ **What Vulnerabilities Can It Detect?**

| Type                     | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **Authorization flaws**  | Checks if low-privileged users can access restricted actions |
| **Authentication flaws** | Checks if unauthenticated users can access protected content |

---

## ⚙️ **How to Use Autorize?**

- Provide the **low-privileged user's cookie** to the plugin.
- Browse the website **normally as a high-privileged user**.
- Autorize will:

  - Monitor your requests.
  - Re-send them using the low-privileged session or no session at all.
  - Compare the responses and flag any discrepancies.

---

## 📊 **Results and Status Colors**

Autorize shows the result of each test with a colored label:

| Status             | Color     | Meaning                                                             |
| ------------------ | --------- | ------------------------------------------------------------------- |
| **Bypassed!**      | 🔴 Red    | Authorization bypass! Low-privileged user could perform the action  |
| **Enforced!**      | 🟢 Green  | Authorization correctly enforced — access denied for low-privileged |
| **Is enforced???** | 🟡 Yellow | Unclear — plugin needs more configuration to verify enforcement     |

---

## 🛠️ **Advanced Features**

- **No setup required**, but customizable for more control:

  - Choose which requests to test or ignore
  - Configure how strict the enforcement check is
  - Save the plugin state
  - Export the test report in **HTML** or **CSV**

---

## 🧪 **Real-World Example — Hospital System**

Let’s say you're testing a hospital web app:

1. You log in as a **Doctor**.
2. You give Autorize the **cookie of a Patient**.
3. You visit `/patients/5/edit` as the Doctor.
4. Autorize re-sends the same request with the Patient's cookie.

   - If the server **responds with the same data or allows the action**, it shows:

     - 🔴 **Bypassed!** — Vulnerability found

   - If access is **denied**, it shows:

     - 🟢 **Enforced!** — Secure

---

</details>

#### **Mozila Firefox** [PwnFox](https://addons.mozilla.org/en-US/firefox/addon/pwnfox/)

- **PwnFox** is a **Firefox browser extension** designed to make **web penetration testing easier**, especially when you are working with **multiple user roles or sessions** during testing.
- It allows you to **run multiple isolated sessions in the same browser window** — each with its **own cookies, headers, and identity** — and visually **color-code** them so you never mix things up.

<details>
        <summary>Summary</summary>

## 💡 **Why is it Useful?**

During web app testing, you often need to:

- Log in as **admin**, **user**, **guest**, or even **attacker** — at the same time.
- Switch between those roles **quickly and safely**.
- Make sure you're sending requests with the **correct session**.

Instead of using multiple browsers, private windows, or external tools...
**PwnFox** lets you do it all in one browser window — clean, fast, and organized.

---

## 🧠 **How Does It Work?**

### 🔑 Key Features:

| Feature                        | What It Does                                                          |
| ------------------------------ | --------------------------------------------------------------------- |
| **Multi-session support**      | Create isolated sessions inside the same Firefox window               |
| **Per-tab session isolation**  | Each tab can have its own set of cookies and headers                  |
| **Color-coded tabs**           | Easily identify which session you’re using (Admin = Red, User = Blue) |
| **Burp integration**           | Works smoothly with Burp Suite via a proxy                            |
| **Custom headers per session** | Add headers like `X-Role: admin` or `Authorization` tokens for APIs   |

---

## 🧪 **Example — Testing a Hospital App**

You're testing a hospital app with these roles:

- 🩺 **Doctor**
- 🧑‍⚕️ **Admin**
- 🧍‍♂️ **Patient**

### With PwnFox:

- **Tab 1** (🔴 Red): Logged in as **Admin**
- **Tab 2** (🟢 Green): Logged in as **Doctor**
- **Tab 3** (🔵 Blue): Logged in as **Patient**

Now, you can:

- Open the same URL in all tabs and compare responses
- Perform actions from one role, and instantly test the same from another
- Spot broken access control (like if a patient can access admin pages)

---

## 🔐 Real-Life Benefits

- Save **tons of time** — no need to log in/out constantly
- Avoid **session confusion** — tabs are isolated
- Great for testing **RBAC**, **horizontal access control**, and **session management**

---

## 🛠️ Bonus: Works Beautifully with Burp Suite

Since PwnFox sends all traffic through **Burp Proxy**, it’s perfect for:

- Capturing and repeating requests
- Using it with extensions like **Autorize**
- Performing **manual and automated testing** side-by-side

---

## 🔚 Summary

| 🔧 Tool      | 🔍 Purpose                                         |
| ------------ | -------------------------------------------------- |
| **Autorize** | Detect **authorization flaws** automatically       |
| **PwnFox**   | Manage **multiple sessions** easily during testing |

---

</details>

## How to Prevent Access Control Vulnerabilites?

#### Preventing Access Control Vulnerabilities

* Use a security-centric desgin where access is verified first and ensure all requests go throgh an access control check.
* Except for public resources, deny access by default.
* Apply the principal of least privilege throughout the entire application.
* Consider using attribute or feature-based access control checks instade role-based access control.
* - NIST Attribute Based Access Control.
* - NIST Special Publication 800-162.
* Access control checks should always be performed on the server side. 
