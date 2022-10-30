1.User Enumeration

Description:

The Green site is vulnerable to user Enumeration in the login session, login with an incorrect username of "John" and incorrect password gives an error of "login was unsuccessful" but login with a correct username of "pperson" and incorrect password gives an error of "login was unsuccessful" in bold.

![User Enumeration-Green](https://user-images.githubusercontent.com/109797939/198902680-5783cec2-fe34-4054-b581-5cf1f8e68499.gif)


2.Sql injection

Description:

The Blue sit is vulnerable to sql injection, the attacker is able to make the database sleep for 5 session with the following payload attacked to the "find a sales paerson" page 
' or sleep(5)='

![sql injection-blue](https://user-images.githubusercontent.com/109797939/198902946-ce5ccd7d-c91a-424d-a364-92505ea6f2e6.gif)
