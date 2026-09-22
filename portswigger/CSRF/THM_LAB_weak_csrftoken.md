In the previous task, we exploited a feature that lacked CSRF protection. In real applications, however, developers often try to defend against CSRF by adding a token to sensitive requests. This is a good step, but only if the token is unique, unpredictable, and properly validated. If the token is generated in a weak or reversible way, an attacker may still be able to bypass the protection.

In this task, we will look at a basic example of reversing weak CSRF protection and then use that knowledge to launch an image-based CSRF attack.

## Practical 

In the attached VM, log in to the StaffHub application at `http://staffhub.thm:8080` using the credentials `user` and `user`. After logging in, navigate to the settings page, where you will find a feature that updates the role of the currently logged-in user.

![The settings page with the "Update Role" option.](https://cdn-images.tryhackme.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1773043120537.png)

This action is protected by a CSRF token. At first glance, this may look secure. If you view the source or review the request and token value, we can see that the token is not randomly generated. Instead, it is derived from predictable user data and can be easily reversed, as it is actually the base64-encoded value of the user's role.

![source code review showing csrf token.](https://cdn-images.tryhackme.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1773045097317.png)

 For example, the application includes a hidden CSRF field like this:

 `<input type="hidden" name="csrf_token" value="YWRtaW4=">`

If we decode this value, we get the value `admin`. You can use [CyberChef (opens in new tab)](https://gchq.github.io/CyberChef/#recipe=From_Base64\('A-Za-z0-9%2B/%3D',true,false\)&input=WVdSdGFXND0) or decode using [this (opens in new tab)](https://emn178.github.io/online-tools/base64_decode.html)website.

This indicates that the application uses a weak, predictable token-generation method. Since the attacker understands how the token is created, they can reproduce it on their own malicious page.

## Preparing the Payload

In the AttackBox, navigate to the web directory using `cd /var/www/html`, create a file using `nano role.html`  and add the following code:

 `<html> <body>  <h2>StaffHub Internal Notice</h2> <p>Move your mouse over the banner below to load the latest role updates.</p>  <img src="http://staffhub.thm:8080/one.png" onmouseover="window.location='http://staffhub.thm:8080/update_role.php?role=staff&csrf_token=YWRtaW4='" width="400">  </body> </html>`

Save the file for sharing with the target using `http://CONNECTION_IP:81/role.html`.

The code above creates a simple webpage that displays an image with a small JavaScript event handler attached. When the user moves their mouse over the image, the `onmouseover` event triggers a redirect to the StaffHub URL that contains the role update request along with the CSRF token. As a result, the victim’s browser automatically sends the request to the application using the victim’s authenticated session cookie, allowing the role change to be performed without the user’s knowledge.

## Sending the Payload

As in the previous case, the attacker can use any social engineering method to send the link to the victim (over email or chat). If the victim is already logged in to StaffHub and visits this page, hovering over the image will cause the browser to request the vulnerable URL. Since the request contains the expected CSRF token and the victim’s session cookie is automatically included, the application will treat it as legitimate.

Switch back to the target VM tab. Then open the link in the same browser where you are logged in to the application, as shown below:

![The vulnerable page created by the attacker.](https://cdn-images.tryhackme.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1775072457287.png)

As a result, on mouse over, the browser will make a call to the vulnerable endpoint with correct parameters like `role` and `csrf-token`, which will change the logged-in user’s role from admin to staff. 

![flag value after demoting the user from admin to staff.](https://cdn-images.tryhackme.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1773046634047.png)

Weak CSRF protection can create a false sense of security. A token must be unique, unpredictable, and securely tied to the user session. If it can be guessed or reversed, it can still be abused by an attacker. In the next task, we will examine the appropriate defensive measures developers should use to prevent CSRF attacks effectively.