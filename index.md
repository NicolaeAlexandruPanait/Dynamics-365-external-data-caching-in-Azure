## Dynamics 365 external data caching in Azure

Throughout my carrer as a Dynamics 365 specialist I was always involved in implementation where an external application was required to connect to the backend solution in real time.
Such scenarios will include usage of a separate e-commerce solution integration or maybe an on premise portal.
Although there is the option to export the data in datalake, it often poses a challenge undestanding the normalized table structure and the data is better queried using data entities.

One possible option to achieve that without having to directly query the dynamics environment is to maintain an external cache for the master data.
This solution involves using the API management with and external redis cache.
This will minimize the OData calls to the backend system and implicitly reducing the risks of throttling. 
Due to the feature link , another benefit that this solution will provide is to have a data layer close to the client application, even on the same network gateway, which will drastically improve the overall user experince.

### Installing the Postman Interceptor chrome extension

Postman Interceptor extenison can be installed in any chrome based browser. 
The only requirement is to have the **Capture cookies** set to ON just like in the setup below



### Postman setup

First we need to setup the postman interceptor to **Capture requests and cookies**

The setup is quite easy and completely automated by the postman team. Once that is completed you will need to specify the url of the dynamics environment.
You should end up with something similar setup.  

![Postman Cookie setup](https://user-images.githubusercontent.com/25058196/158826075-5d0912f1-1576-46a6-a4f8-e71f45f7cb71.PNG)


### API testing

Now that all setup is in place let's do an simple OData call. One thing to note is that the D365 url must be setup without the trailing backslash.

Here is a simple example which creates a vendor group.

![Postman OData call](https://user-images.githubusercontent.com/25058196/158974766-8aea6643-162d-4ddd-bc93-79d5102f762c.PNG)

Simply run it without any additional setup and the record will get created

![Postmand Odata result](https://user-images.githubusercontent.com/25058196/158975540-650f827c-e172-4361-984d-8c697a455a8c.PNG)

### Observations

There are other options out there that can help the developers such as Power automate connectors. However the Dynamics connectors are premium.

There is also the option to *Edit and resend the request* in the Mozilla Firefox. Thanks to this feature i was able to work out that the "Origin" header needs to be added to the Postman call.

I hope this will make the work easy for some of the integration experts out there. 

Happy testing !!!

