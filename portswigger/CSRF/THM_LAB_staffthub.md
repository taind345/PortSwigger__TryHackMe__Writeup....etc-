Now that we understand where CSRF vulnerabilities appear, it is time to see how attackers actually exploit them. In this task, we will perform a basic CSRF attack by forging a malicious HTML form.

## Practical

In the attached VM, visit the StaffHub app at `http://staffhub.thm:8080` and log in with the credentials `user` and `user`. Please use the mentioned domain name instead of accessing the web app using the IP address. Once you are logged in, visit the settings page, where you will find the option to change the email, as shown below:

![The settings page with the "Update Email" option.](https://cdn-images.tryhackme.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1772918648421.png)

The **StaffHub** employee portal allows users to update their email address from the account settings page. When a user submits the form, the browser sends a POST request to the server containing the new email address. However, the application lacks CSRF protection, meaning the server does not verify the origin of the request. Here is the HTML code that posts the content:

 `<form action="update_email.php" method="POST">     <div class="input-field">         <i class="material-icons prefix">alternate_email</i>         <input id="email" type="email" name="email" required>         <label for="email">New Email Address</label>     </div>      <button class="btn waves-effect waves-light teal btn-rounded" type="submit">         Update Email         <i class="material-icons right">save</i>     </button> </form>`

Because of this, an attacker can recreate the same request from another webpage and trick the victim’s browser into automatically submitting it.

Notice that the request only contains the email parameter. There is no additional value, token, or verification mechanism to confirm that the request came from the legitimate website. This means that anyone who knows the structure of this request could reproduce it elsewhere.

## Crafting a Malicious Page

An attacker can create a malicious webpage containing a hidden form that submits the same request to the StaffHub server. The form can be automatically submitted using JavaScript when the victim opens the page. Switch to your AttackBox tab and navigate to the `/var/www/html` using the `cd /var/www/html` command, and create a new file using `nano settings.html` and copy the following code:

 `<html> <body>  <form action="http://staffhub.thm:8080/update_email.php" method="POST" id="attack"> <input type="hidden" name="email" value="attacker@evilmail.thm"> </form>  <script> document.getElementById("attack").submit();  // redirect user after the request is sent setTimeout(function() {     window.location.href = "http://staffhub.thm:8080/settings.php"; }, 1000); </script>  </body> </html>`

This page can be accessed via the link `http://CONNECTION_IP:81/settings.html`. If a victim who is already logged in to StaffHub visits this page, their browser will automatically send the forged request to the server along with their session cookie. From the server’s perspective, the request appears to come from the authenticated user.

The attacker can send the URL to the target using any social engineering technique, such as convincing the target to click the link in a chat or email.

Switch back to the target VM tab. Then open the link `http://CONNECTION_IP:81/settings.html` in the same browser and same tab where you are logged in to the application and it will update your email to [`attacker@evilmail.thm`](mailto:attacker@evilmail.thm) as shown below:

![flag after updating the email.](https://cdn-images.tryhackme.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1772922823745.png)

## What Exactly Happened?

Since the application does not verify the request’s origin, the server processes it normally and updates the user’s email address to the attacker’s value. This simple technique demonstrates the core idea behind CSRF attacks: forcing an authenticated user’s browser to perform actions without their knowledge.

In the next task, we will see how a weak token implementation of CSRF can cause the victim’s account information to change automatically.