---
date: '2022-11-21'
thumbnail: ./assets/hero.jpg
title: Implementing Jenga Account Balance API using Node.js
description: Implementing Jenga Account Balance API on a RESTful Node.js API.
category: backend
author: kennedy-mwangi
---


### Pre-requisites

To continue in this article, it is helpful to have the following:

- [Node.js](https://nodejs.org/en/) installed on your computer.

- [Jenga](https://www.jengaapi.io/) API Account.

- [Postman](https://www.postman.com/) installed on your computer.

- Basic knowledge working with JavaScript.

### Overview.

- [Generating the public key and private key](#generating-the-public-key-and-private-key)

- [Setting up the application](#setting-up-the-application)

- [Configuring the signature](#configuring-the-signature)

- [Account Balance Query](#account-balance-query)

- [Send Mobile Money](#send-mobile-money)

- [Conclusion](#conclusion)

### Generating the public key and private key

- Proceed to your preferred working directory.

- Run the following command to generate a public key and a private key.

    ```bash
    openssl genrsa -out rsa.private 1024
    ```
    
- From your Jenga API dashboard:

    - Under *settings*, *API Keys* add your public key.

    - Click on Generate New Keys which will output a *Merchant Code*, *Consumer Secret*, *API Key*. 
#### Setting up the application

- From your working directory, Initialize the node js application by running the below command:

    ```bash
    npm init -y
    ```

- Install the following packages:

    - *express* : For creating a web server.
    - *axios* : For sending HTTP requests.
    - *nodemon* : For starting and restarting the development server.
    - *dotenv*: For configuring and loading environmental variables.

    ```bash
    npm i express axios dotenv
    ```

    ```bash
    npm i --save-dev nodemon
    ```

- Update the *package.json* as below to include the script to start the development server:

    ```js
    {
        "dev":"nodemon app.js"
    }
    ```

- In the project root directory, create an *.env* file. Add the following:

    ```js
    PORT = 5000
    JENGA_API_BASE_URL = "https://uat.finserve.africa"
    MERCHANT_CODE = "your_jenga_merchant_code"
    CONSUMER_SECRET = "your_jenga_consumer_secret"
    API_KEY = "your_jenga_api_key"
    ```

- Create the main application file:

    ```bash
    touch app.js
    ```

- In *app.js*:

    - Import the necessary packages:

        ```js
        const express = require('express');
        const axios = require('axios');
        require('dotenv').config();
        ```

    - Initialize the express application:

        ```js
        const app = express();
        app.use(express.json()); // for receiving json data.
        app.use(express.urlencoded({extended:true})); // for urlencoded data
        ```

    - Define the routes:

        ```js
        app.get("/",(req,res,next) => res.json({
            success:true,
            message:"Hello World!"
        }));
        ```

    - Start the application:

        ```js
        app.listen(process.env.PORT, () => console.log(`App started on port ${process.env.PORT}`));
        ```

#### Configuring the signature

- In the *app.js* file:

    - Import the crypto core module:

        ```js
        const crypto = require('crypto');
        ```

    - Define the following function to generate the signature:

        ```js
        function generateSignature(plainText="")={
            const privateKey = fs.readFileSync(join(__dirname,'privatekey.pem'), 'utf8'); // path to your private key
            return crypto.createSign('sha256').update(plainText).sign({
                format: 'pem',
                key: privateKey,  
            }, 'base64');
        }
        ```

#### Account Balance Query

- In *app.js*:

    - Import the axios package:

        ```js
        const axios = require('axios');
        ```

    - Configure a route for account balance:

        ```js
        app.post('/account-balance', (req,res) => {
            // Rest of the logic...
        });
        ```

    - Inside the route:

        - Extract the payload:

            ```js
            let {countryCode , accountNumber } = req.body;
            ```

        - Getting the OAUTH token:

            ```js
            try {

                let authToken = await axios.post('https://uat.finserve.africa/authentication/api/v3/authenticate/merchant',{
                'merchantCode':process.env.MERCHANT_CODE,
                'consumerSecret':process.env.CONSUMER_SECRET
                },{
                headers:{
                    'Api-Key':process.env.API_KEY,
                    'Content-Type':'application/json'
                }
                });
                
                authToken = authToken.data;
                
            }catch(error){
                throw error;
            }
            ```

        - Getting the signature:

            ```js
            let signature = generateSignature(`${countryCode}${accountNumber}`);
            ```

        - Getting the account balance:

            ```js
            let response = await axios.get(
                `${process.env.JENGA_API_BASE_URL}/v3-apis/account-api/v3.0/accounts/balances/${countryCode}/${accountNumber}`,
                {
                headers: {
                    Authorization: `Bearer ${authToken.accessToken}`,
                    signature: signature,
                },
                }
            );
            ```

        - Return the response:

            ```js
            return res.status(200).json({
                success: true,
                message: response.data,
            });
            ```

To test the functionality:

- Ensure the development server is running:

    ```js
    npm run dev
    ```

- Launch Postman.

- Send a *POST* request to */account-balance*. Your JSON payload, should be similar to:

    ```js
    {
        "countryCode":"KE",
        "accountNumber":"1450160649886"
    }
    ```

- If everything is OK, your response should be similar to:

    ```js
    ```


#### Conclusion

To gain more insights on the technologies used in this article, below are resources:

- [Jenga API docs](https://developer.jengaapi.io/docs)
- [Express Docs](https://expressjs.com/en/5x/api.html)
