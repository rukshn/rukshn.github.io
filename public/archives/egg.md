Last week government launched its online covid tracker called the **["StaySafe"](https://staysafe.gov.lk) program.**

The program is similar was launched in China, a QR code-based system. Where people should scan a QR code when they enter a location, a government server will save and track their check-in and check-out depending on the QR code scanning.  

Leaving the privacy aspect aside, since we are experiencing a second wave of COVID 19 a tracking solution can help to identify people exposed to a potential COVID 19 patient.  

I have not investigated an exploit for a very long time. I was busy with my daily job, and my master's in Health Informatics.

However, a [Facebook post](https://www.facebook.com/nalina.kanchana/posts/3724152284301902) about someone finding unofficial API access to check the infective status of any person just passing their NIC number (it's like a social security number) caught my eye.

![The facebook post that started it all](/images/posts/2020/staysafe/123760394_3724198324297298_1148737849357766572_n.jpg)
*The facebook post that started it all*

The post showed an unofficial API, where anyone can just type a NIC number and get their covid status, without requiring any authentication.

Since this is all about to do with a Health Informatics system, it sparked my curiosity to learn more about this StaySafe platform, potentially finding an exploit and ultimately breaching it.

## Step 1

I visited the stay safe website, and at a glance, it has a simple landing page.

The first thing I did was to sign up as a new user. 

According to the landing page, I had to do is text my NIC number to 1919, or scan a QR code using my mobile.

- If I send a text message, they will create a new account, associate my NIC number with my mobile number.

- If I scan the QR code, it will take me to a signup form, where I have to give my NIC number, mobile phone number, and they will signup me as a new user using the same details. 

So I picked up my phone and sent 123456789V as my NIC number to 1919. I was hoping the system will reject my NIC number since this is obviously an incorrect NIC number (just like a credit card number, an NIC number follows a standard pattern).

However, I was able to sign-up for the system using a fake NIC number, which means I can game the system, or impersonate someone else, using someone else's NIC number.

![I was able to register with a bogus NIC](/images/posts/2020/staysafe/IMG_8B243A37279C-1.jpg)
*I registered with a fake nic*

## Step 2

  Still, I was not able to find information about the unofficial API as I saw on the screenshots.

I scanned the QR code using my phone, and it took me to a registration form.

**[https://staysafe.gov.lk/customerSignUp/00047](https://staysafe.gov.lk/customerSignUp/00047)**

The system generates different forms for different locations, and each location will have its own unique URL.
  
The forms also had the same vulnerability, where I can signup using a fake NIC number.

I signed up through the form using fake data, and at the same time checked the network-requests.

The requests are all made using javascript, Iooked the developer tools, and looking at the requests I saw that anyone could make a get request to an unofficial API endpoint to get the infected status of the covid status just sending their NIC number with the get request.

![The unofficial API to check anyone's infection status](/images/posts/2020/staysafe/Screenshot-2020-11-10-at-14.34.27.png)
*The unofficial API to check infected status*

## **Step 3**

I made a simple Google search.

**Search site:staysafe.gov.lk**

Looking at the Google results, I was able to access a backend dashboard for merchants (location owners) that was not supposed to be accessible to users. 

![Merchant dashboard](/images/posts/2020/staysafe/Screenshot-2020-11-10-at-14.37.20.png)
*Merchant dashboard*

And this page was not protected by any authentication. However, I was unable to go to click links because they are redirecting me to the landing page.

So again looking at network requests, I was able to find that they are checking whether the user is logged in from the front end.

By using javascript and a merchant ID is sent to the API, it supposedly checks whether that merchant ID exists in the database (which was a 4 digit ID), if the merchant ID exists, a valid response is sent, if it does not exist an invalid response is sent, and the user is redirected.

Looking at their javascript code, I was able to find that they save the merchant ID in the browser local storage. So by using the developer tools, I created a 4 digit merchant ID on the browser local storage. (I was able to obtain a valid merchant ID from the signup form.)

After refreshing the page, I was able to access the other broken features on the dashboard. I was able to submit some forms, but I was not able to get any valuable data through that dashboard. And I didn't know whether the data I passed made any difference.

![I gained more privilates after setting the localstorage object](/images/posts/2020/staysafe/Screenshot-2020-11-10-at-14.38.20.png)
*Gained more privilages in the merchant dashboard*
  
## Step 4

Then I saw another photo on the same Facebook thread that showed a password field.

And I followed that URL - staysafe.gov.lk/admin

Then I came to a page with the password field, saying I need some secret key to log in to the next page.

![Login page I found on the Facebook post](/images/posts/2020/staysafe/123849698_3724191044298026_8139327141724236291_n.jpg)
*Login page I found in the same Facebook post*

I entered some generic passwords to check whether it will open, but for no luck.

Again I checked the page source code. All the user checks are done in front-end using javascript.

So I went through the javascript code and came across this unusual function, **function saveAdmin() {}**

    function saveAdmin() {
	    var fname = document.getElementById("sfname").value;
		var lname = document.getElementById("slname").value;
	    var contact = document.getElementById("sphone").value;
	    var email = document.getElementById("semail").value;
		var password = document.getElementById("spassword").value;
	    var isOk = true;
    
	    if (isEmpty(email)) {
		    setErrorMessage('semail', 'Email is required.')
		    isOk = false;
        } else {
		    removeErrorMessage('semail');
	    }
    
	    if (isEmpty(contact)) {
		    setErrorMessage('sphone', 'Contact Number is required.')
		    isOk = false;
	    } else {
		    removeErrorMessage('sphone');
	    }
    
	    if (isEmpty(password)) {
		    setErrorMessage('spassword', 'Password is required.')
			isOk = false;
	    } else {
		    removeErrorMessage('spassword');
	    }
    
	    if (isOk === false) {
		    return;
		}
    
		var form = new FormData();
	    form.append("mainTypeId", "1");
	    form.append("mainType", "Admin");
	    form.append("subTypeId", "9");
	    form.append("subType", "stay safe");
	    form.append("contactNumberTypeId", "2");
	    form.append("contactNumberType", "Personal");
	    form.append("contactNumber", contact);
	    form.append("email", email);
	    form.append("fname", fname);
	    form.append("lname", lname);
	    form.append("password", password);
    
	    var settings = {
		    "url": main_api_url + "/admin/save_admin",
		    "type": "POST",
		    "timeout": 0,
		    "headers": {
				 'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')    
		    },
		   "processData": false,
		   "mimeType": "multipart/form-data",
		   "contentType": false,
		   "data": form,
		   success: function (data) {	
		   Swal.fire({
			   // position: 'top-end',
			   icon: 'success',
			   title: 'Account Has Been Created'
		   }).then(function () {
			   window.location.replace(main_url + '/admin/dashboard');
		    });
		   },
			error: function (data) {
				console.log(data);
			}
		};
	    $.ajax(settings);
    }

*The javascript code*

Further inspecting this function's code, I was able to figure out that all I need to do is pass a name, email, a phone number, a password, and a CSRF token (which I can collect from the website) as a POST request to an API URL which was available in the saveAdmin function.

And looking at the function, it looked as if it was successful, I will be able to create an administrator account on the system, which can be the vulnerability I was looking for.

I made a simple javascript fetch request with all the required fields.

And as a response, I got a successful response, which meant I just created a new Admin account. I was surprised and excited at the same time.

Creating an admin account is one thing, but what about login? I went back to the login page and tried entering the password I just passed in the previous POST request. It didn't work.  

## The breakthrough

Again looking at the javascript code, just as what they did with the merchant dashboard, I saw that they are checking the authentication from the frontend using javascript.  

They store a local storage object with name, id, and email, and upon saving the objects in the local storage, they will redirect you to the dashboard URL, which was also available in the javascript code.

They will make an API call with administrator details, ID, name, and email. And my theory was that if the data exists in the backend, then the API will return a valid response, and if not it will return a response.

Since I made an admin account, and I had the email, name with me, and received my admin ID from the response of the saveAdmin function, I just saved these objects in the local storage, crossed my fingers, and typed the dashboard URL on my browser address bar.  

And just like that, I was inside the dashboard of the corona tracker designed to track patient exposure in the country.

## Exploring the national COVID tracker
After a brief moment of joy, I thought of exploring the system to see the functionality of the system.

I could search for anyone who signed up for the system, using their NIC or mobile number.

![StaySafe dashboard](/images/posts/2020/staysafe/Screenshot-2020-11-10-at-00.31.00.png)
*The national covid tracker staysafe dashboard*

I can see the places they have visited, and even mark them as infected, I didn't tamper or saved any data. My intention was only to find the vulnerability and report it.

For some wired reason, when someone logged in to the dashboard, all the registered user data was fetched in a single JSON response to the dashboard. I was able to see that the request returned around 8000+ user data, which included, their mobile phone number, NIC number, and their COVID status.

I'm not sure whether this response included all the user data in the system.

![json request with 8000+ personal data](/images/posts/2020/staysafe/blur-data.jpg)
*The request resposne with 8000+ userdata*

I didn't tamper with any data. I contacted some developers who commented on the original Facebook post that started all this to find out whether this was the only vulnerability.

![I got to see more data than I expected](/images/posts/2020/staysafe/Screenshot-2020-11-10-at-00.34.46.png)
*More info on dashboard*

 But some fellow developers showed me that the website's SSL is not properly configured and you can find out the IP address of the server, which is also found to be correct.

![SSL settings exposed the original IP](/images/posts/2020/staysafe/PHOTO-2020-11-10-09-47-46.jpg)
*SSL vulnerability with the exposed IP address*

## Learning points
-   Even though javascript can give a nice user experience, there is a risk of passing authentication details through javascript. A session value can easily read by the server-side than passing the session details in a javascript request.
-   Use a cookie-based or token-based authentication system.
-   Never trust user input, always validate the data.  

## Status update
- I contacted the CTO of ICTA Mr. Hiranya Samarasekera, I informed about the vulnerability and offered my support to fix the vulnerabilities.

- Mr. Hiranya Samarasekera said that the project was developed by a group of volunteers and offered to ICTA for free.
- The API, admin page, and dashboard are now removed and returns a 404 error.
- ICTA has thanked me for informing the vulnerability and said they are working on a fix.

## More updates
- After writing this Mr. Hiranya Samarasekera contacted me and me and the developers of StaySafe had a discussion where I explained to them about the vulnerabilities we found, I offered my ideas on how to fix it.

- The developers have patched the vulnerabilities and working on a fix.

## Disclaimer
I did not tamper or save any data, I hope I'm the first to find the vulnerability, and no one was able to breach the system and harvest the user data.

The idea of this post is for learning purposes only on finding vulnerabilities and designing a secure system. I take no responsibility for any damage caused by someone wrongfully using this information.