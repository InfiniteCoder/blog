---
date: "2017-12-22T00:00:00Z"
published: true
title: How to authorize using OAuth 2.0 in Postman
---

If you develop APIs, you surely must have heard of [Postman](https://www.getpostman.com/). It is a tool that is used to test API endpoints.  
When authorizing, for APIs protected with Basic Auth, you can simply add the username and password as a parameter to an HTTP call. But when using an external OAuth provider such as Google, one can't simply put the login link and authorize.   
This post will explain how you can authorize for an API using an OAuth 2.0 provider (such as Google) in Postman.

## Authorization
Initially, go to **Authorization** and in the **type** option, select **OAuth 2.0**. In the right part of the window, you shall see the _access token_ section, which lists the current access token being used (which would be empty for now). Click on the **get new access token** button.
![postman main window](/img/Postman-main-window.png)

You shall see the access token window, where you have to add details such as client id and client secret. You can get these from your OAuth provider. Insert the relevant details and click **request token**.
![postman access token window](/img/postman-token-window.png)

Now, postman launches another window, which is basically a lightweight browser, which will open the login page of your OAuth 2.0 provider. After you **login**, Postman will retrieve the access token, and display the details to you.

You can notice that the access token is now shown in the right part of the postman window.
![access token added](/img/access-token-added.png)

That's all, now you can use this token for other requests to your API endpoints.

## Refreshing tokens
To refresh the token, click on **get new access token**. It will show the access token window, where the details should be already added. Simply click on the **request token** button, and the access token shall be refreshed.

## Log in as different user
Postman stores the cookies🍪 even after you close and relaunch it, so you will still be logged in as the same user on every session. If you want to log in as a different user, you need to delete the cookies. To do this, click on the cookies🍪 link in the upper right part of the window, and delete all cookies from your OAuth provider.  
![postman cookies window](/img/postman-manage-cookies.png)
Now repeat the previous process to get the token. This time, Postman shall show you the login page again.
