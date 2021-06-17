# **Chat Bot API** V1

## order
* [Overview](#Overview)
* [Create partner account for chatbot](#for chatbot-partner-account-create)
* [How to write `Webhook`](#webhook-write-how)
  * [`Webhook` requirement](#webhook-requirement-requirement)
  * [**[STEP 1]** Receive Event from Naver Talk Talk](#step-1-Receive Event from Naver Talk Talk)
  * [**[STEP 2]** Create echo server (chatbot example)](#step-2-echo-server chatbot-example-create)
* [How to write `send API`](#send-api-write-method)
* [event specification](#event-spec)
  * [Event Basic Structure] (#Event-Basic-Structure)
  * [`open` event](#open-event)
  * [`leave` event](#leave-event)
  * [`friend` event](#friend-event)
  * [`send` event](#send-event)
  * [`echo` event](#echo-event)
  * [`action` event](#action-event)
  * [`persistentMenu` event](#persistentmenu-event)
* [message type specification] (#message-type-spec)
  * [textContent](#textcontent)
  * [imageContent](#imagecontent)
  * [compositeContent](#compositecontent)
    * [`CompositeContent` Object](#compositecontent-object)
    * [`Composite` Object](#composite-object)
    * [`Image` Object](#image-object)
    * [`ElementList` Object](#elementlist-object)
    * [`ElementData` Object(`LIST` type)](#elementdata-objectlist-type)
    * [`Button` Object](#button-object)
    * [Notes when using Composite messages](#Composite-messages-when-using-notes-notes)
  * [Quick Button] (#Quick-Button)
* [error-spec](#error-spec)
* Extended API
  * [Profile API V1](/profile_api_v1.md)
  * [Pay API V1](/pay_api_v1.md)
  * [Image Upload API V1](/imageupload_api_v1.md)
  * [Handover API V1](/handover_v1.md)
  * [Product Message API](/product_message_api.md)
  * [Plugins (BETA)](/plugins.md)
* [UI component](/ui_component_v1.md)
  * [TIME component](/time_component_v1.md)
  * [CALENDAR component](/calendar_component_v1.md)
  * [TIME-INTERVAL component](/time_interval_component_v1.md)
<br>
<br>

## summary
The NAVER Talk Talk chatbot platform provides the Chat Bot API, which provides the necessary tools to create a bot on NAVER Talk Talk. You can use the Chat Bot API to send various messages to users and connect Naver services such as Naver Shopping and Pay to the bot.

The following figure shows the working structure of the Chat Bot API.
![composite_message](/images/chatbotapi_structure.png)
* Various user events are delivered to the chatbot as a 'Webhook' through the 'Chatbot Platform'.
* Chatbots can send message delivery events to users using the ‘Send API’.
* ‘Chatbot Platform’ provides ‘Profile API’, ‘Pay API’ as well as basic ‘Message API’.
<br>
<br>


## Create partner account for chatbot
To use the Chat Bot API, you must first create a partner account for your chatbot. To create a partner account for your chatbot:
1. Access [Naver Talk Talk Partner Center] (https://partner.talk.naver.com/).
2. Log in with a company 'group ID' or a representative 'personal ID'.
3. Click **Manage My Account > Create a new Talk Talk account**.
4. In **Choose a service**, without selecting a service, click **Connect service later**.
5. Select **Individual** to create a test account, select **Domestic Business**, **Overseas Business** or **Institution/Organization** to create a service account, then **Next** Click .
6. Enter information such as **Representative image**, **Profile name**, **Mobile phone number**, and click **Apply for use**.
 The created account will be displayed as **under review** status, and when the review is completed, it will change to **in use** status and a notification will be sent via SMS.
7. From the **Registered Accounts** list in **My Account Management**, click **Account Management** to enter the Account Home.
8. Click **Chatbot API Settings** under **Developer Tools** to apply, agree to the Terms of Use and set up the chatbot.
<br>

> **Processing exclusions from search during testing**<br>
>
> If you insert ‘[Test]’ in front of the profile name in **Partner Center> Basic Settings> Profile Information**, it will be excluded from ‘Find Talk Talk Merchant’. This will allow you to block user visits from search during testing.<br>
> (Example) `Talk Talk Chatbot` -> `[Test] TalkTalk Chatbot`

<br>

## How to write `Webhook`
A webhook is used to deliver user events to the chatbot. How to write `webhook`<br>

### `Webhook` requirements
* `Webhook` must support `TLS` based communication.
  * Messages between Naver Talk Talk and external services that work with each other are transmitted in plain text without encryption, so they must be protected.
  * TLS supported is `TLSv1`, `TLSv1.1`, and `TLSv1.2`.
  * Only valid certificates issued by a formal certification authority can be used.
* If you need to register ACL to use `Webhook`, add the following IP address list.
  * 117.52.141.192/27 (117.52.141.193 ~ 117.52.141.222)
* When calling `Webhook` in Naver Talk Talk, the setting values ​​are as follows.
  * Connection timed out: `3 seconds`
  * Read timed out: `5 seconds`
<br>

### **[STEP 1]** Get events from Naver Talk Talk

#### 1. Write a simple code to receive events from Naver Talk Talk.
##### Example code
[`node.js` sample]
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json());
app.post('/', (req, res) => {
  console.log(req.body);
  res.sendStatus(200);
});

app.listen(8080);
```
<br>

[`Spring Boot` Sample]
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableAutoConfiguration
public class BotApplication {

  @PostMapping
  public void event(@RequestBody String body) {
    System.out.println(body);
  }

  public static void main(String[] args) throws Exception {
    SpringApplication.run(BotApplication.class, args);
  }
}
```
<br>

[`php` sample]
```php
<?php
  $json_str = file_get_contents('php://input');
  $json_obj = json_decode($json_str);
  print_r($json_obj);
?>
```
<br>

#### 2. Test access on localhost.
```bash
curl -X POST -H "Content-Type: application/json;charset=UTF-8" -d '{ "event": "test" }' "http://localhost:8080/"
```
* After testing on localhost, you should be able to access https://your.domain/ using your official certificate.
<br>

#### 3. Register the URL.
* In **Partner Center > Developer Tools > Chatbot API Settings**, register the URL created above in the **Event Receiving URL** in the **Webhook** area.
<br>

#### 4. The link is completed and you receive an event from Tok Tok.
* Access https://talk.naver.com/{partner ID (eg wc1234}) and try to generate an event as a user.
<br>

* [Event log when entering chat window]

    ```javascript
    {
      "event": "open",
      "user": "al-2eGuGr5WQOnco1_V-FQ",
      "options": {
          "inflow": "list",
          "referer": "https://talk.naver.com/",
          "friend": false,
          "under14": false,
          "under19": false
      }
    }
    ```

* [Event log when sending `hello world` message to chat window]

    ```javascript
    {
      "event": "send",
      "user": "al-2eGuGr5WQOnco1_V-FQ",
      "textContent": {
          "text": "hello world",
          "inputType": "typing"
      }
    }
    ```
<br>

> **Note**
>
> If the event is not received through `Webhook`, select the event you want to receive in **API Settings > Webhook > Select Event**.<br>

<br>

### **[STEP 2]** Create an echo server (chatbot example)
Create an echo server with the following functions:
* Determines various events sent by Naver Talk Talk and replies with appropriate messages.
* Reply to user message with echo message.

#### 1. Write the echo server code as follows.
[`node.js` sample]
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json());
app.post('/', (req, res) => {

  // request logging
  console.log(req.body);

  // default response
  let response = {
    event: 'send', /* send message */
    textContent: {
        text: ''
    }
  };

  switch(req.body.event) {

    // handle the message sending event
    case 'send' :
      if(req.body.textContent) {
        // Send as echo for messages sent by the user
        response.textContent.text = 'echo: ' + req.body.textContent.text;
        res.json(response);

      } else {
        // Otherwise, no response
        res.sendStatus(200);
      }
      break;

    // Handle the chat window open event
    case 'open' :
      switch(req.body.options.inflow) {
        // When incoming from chat list
        case 'list' :
          response.textContent.text = 'You clicked from the list to visit.';
          res.json(response);
          break;

 // when fetched from a link
 case 'button' :
 response.textContent.text = 'You clicked a button to visit.';
          res.json(response);
          break;

 // when there is no funnel
 case 'none' :
 response.textContent.text = 'Welcome to your visit';
          res.json(response);
          break;

        default:
          res.sendStatus(200);
      }
      break;

    // handle friend event
    case 'friend' :
      if(req.body.options.set == 'on') {
        // when adding friends
        response.textContent.text = 'Thank you for being my friend.';
        res.json(response);

      } else if(req.body.options.set == 'off') {
        // When withdrawing a friend
        response.textContent.text = 'Please be sure to add me as a friend next time.';
        res.json(response);
      }
      break;

    // No response to other events
    default:
      res.sendStatus(200);
  }

});

app.listen(8080);
```
<br>

#### 2. Access https://talk.naver.com/{partner ID} to check various events and learn.
* Look at the request log and think about which events occur and how to respond to each event.
<br>
<br>

## How to write `send API`
Usually, a simple chatbot can be implemented with just a 'Webhook'. However, if you need to push a message to the user according to a state change or an event, rather than simply answering the user's question, you need to use the 'Send API' to send an event from the chatbot to Talk Talk.
<br>

Test the API call as follows:

[Request]
```bash
curl -X POST \
    -H "Content-Type: application/json;charset=UTF-8" \
    -H "Authorization: ct_wc8b1i_Pb1AXDQ0RZWuCccpzdNL" \
    -d '{ "event": "send", "user": "al-2eGuGr5WQOnco1_V-FQ", "textContent": { "text": "hello world" } }' \
    "https://gw.talk.naver.com/chatbot/v1/event"
```
* The authentication key (`Authorization`) can be obtained by clicking **Generate** in **API Settings > Send API**.
* If the authorization key (`Authorization`) has been published, you can change the authorization key by clicking **Reset**.
<br>

[Response]
```javascript
HTTP/1.1 200 OK

{
    "success": true,
    "resultCode": "00"
}
```
* For the response result, refer to [`Error specification`](#Error-Specification).
<br>
<br>

## event specification


### Event Basic Structure
The basic structure of an event is as follows:

[`Request` message structure]
```javascript
POST / HTTP/1.1
Host: your.bot.co.kr
Accept: application/json
Content-Type: application/json;charset=UTF-8

{
    "event": "", /* event name */
    "options": {
        /* additional attributes */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* user identification value */
}
```

* ‘event’ contains ‘event name’ to identify various events.
* Additional properties that cannot be explained by specific events alone can be obtained using `options`.
* A 1:1 conversation in Talk Talk is a conversation between a 'partner' and a 'user'.
  * Chatbot corresponds to ‘partner’, and Naver users who use chatbot correspond to ‘user’.
  * `"user": "al-2eGuGr5WQOnco1_V-FQ"` is a user identification value and is an unchanging absolute value corresponding to a specific `Naver ID`.
<br>
<br>

[`Response` message structure]
```javascript
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

/* Used to send an event at the same time as the response */
{
  "event": "send",
  "textContent": {
      "text": "hello world"
  }
}
```

* The response to event delivery in the chatbot must be `HTTP Status` `200` to be considered a success. If 'HTTP Status' is any value other than '200', 'Tok Tok' considers it a failure and logs an error.
* If you want to send an automatic response message to an event, include `Content-Type: application/json;charset=UTF-8` in `Header` and add the event you want to deliver to `body`. In this case, the `user` attribute is ignored and the event is passed to the user who sent the event.
<br>

> **Note**<br>
>
> There are 'synchronous'/'asynchronous' message transmission in the form that the chatbot replies to the user's message.
> * 'Synchronous' is a method of giving a response message to the user's message event. 'Synchronous' can be used if it is a single response or if the resulting message can be sent within 5 seconds.
> * 'Asynchronous' is a method to directly respond to the user's message event ('200 OK') and then directly send the event to Talk Talk using the 'Send API' 'Asynchronous' can be used when a response needs to be delivered in multiple messages, or when a message is sent after a certain amount of time has elapsed.

<br>


### `open` event
When a user enters the chat window, an ‘open’ event is sent along with the incoming information.
* How was the inflow information using `inflow`?You can identify the URL of the incoming page using `referer`.
* If you use a separate `from` parameter, you can use `from` to receive a value.

[All Requests]
```javascript
{
    "event": "open",
    "options": {
        "inflow": "button",
        "referer": "http://storefarm.naver.com/pqbdo/products/309672359",
        "from": "309672359",
        "friend": false, /* false: not friend, true: friend */
        "under14": false, /* false: over 14 years old, true: under 14 years old */
        "under19": false, /* false: over 19 years old, true: under 19 years old */
"unreadMessage": true /* Whether there are unread messages among the messages sent by the partner or chatbot, false: No unread messages, true: There are unread messages */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* user identification value */
}
```

[‘options’ if it was entered through a button or link]
```javascript
    "options": {
        "inflow": "button",
        "referer": "http://storefarm.naver.com/pqbdo/products/309672359",
        "from": "309672359"
    }
```

[‘options’ in case of inflow to chat chat list]
```javascript
    "options": {
        "inflow": "list",
        "referer": "https://talk.naver.com/"
    }
```

[`options` if the URL was entered by directly entering the URL in the browser address bar]
```javascript
    "options": {
        "inflow": "none"
    }
```

> **Note**
>
> Funnels can provide a lot of information when talking to users.
> * If it is a shopping mall, if it comes from 'http://shopping.com/product/1234', it can respond to the user's intention by asking, 'Do you want to purchase 1234 products?'.
> * In some cases, if a button is exposed along with multiple products on a single web page, it is not possible to distinguish the products with only `referer`. In that case, you can use the `from` parameter to get the product number like `https://talk.naver.com/wc1234?from=1234` and `https://talk.naver.com/wc1234?from=5678`. can.
> * Since Talk Talk is a web service, you can initiate a conversation with the chatbot via a button or link on any web page. If you run banner ads for multiple services, you can use it as statistical data to determine which banners users mainly come to and which banners to focus on.
> * In the case of `"inflow": "list"`, it is impossible to know if it was `open` just to view the past conversation history due to inflow through the chat chat list, so you must respond appropriately according to the time or context of the last message.
> * `under14` is a subproperty of `options` that must be included in the `open` event. When you agree to the terms and conditions for personal information collection, you must obtain the consent of a 'legal representative' if you are under the age of 14 ('Personal Information Protection Act Article 22') If the chatbot collects personal information, implement the 'legal representative' consent process or Children under the age of 14 must be instructed not to use it.

<br>

### `leave` event
This event occurs when the user presses **Exit** in the chat window or chat list.
* Even if you put `request` in `Response` of `leave` event, no message is sent and it is always ignored.
* The 'leave' event leaves the 'chat window' like a normal messenger experience and does not exist in the 'chat list', but when the chatbot sends a message back to the user, the chat room is recreated and displayed in the 'chat list'.
* The 'leave' event allows the chatbot to choose whether to actively send subsequent messages to the user or stop talking at this point.
* Just because a user has left the 'chat window' doesn't mean they don't want to talk to the chatbot.


```javascript
{
    "event": "leave",
    "user": "al-2eGuGr5WQOnco1_V-FQ"
}
```
<br>
<br>

### `friend` event
This event occurs when the user clicks **Add Friend** or **Remove Friend**. You can distinguish between 'Add friend' and 'Withdraw friend' by setting `set` in `options` to `on`/`off`.

```javascript
{
    "event": "friend",
    "options": {
        "set": "on" /* on: add friend, off: withdraw friend */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ"
}
```
> **Note**
>
> We recommend that you turn off event reception or do not respond to `friend` events even if they are received. In Talk Talk, a friend means a person to whom you can send a message using **Marketing Management > Group Message** in [Partner Center] (https://partner.talk.naver.com/). It is recommended to send various messages by setting **Marketing Management > Friendship Thank You Message** rather than having the chatbot directly send a thank you message for **Add friend**.

<br>

### `send` event
The `send` event is an event that is fired when a message is sent. The `open`, `leave`, and `friend` events mentioned earlier can only be fired by the `user`, but the `send` event can be fired by both the `user` and the `partner`.<br>
The `send` event provides the following message types.
  * `textContent`: text message
  * `imageContent`: a message consisting of a singular image
  * `compositeContent`: a composite message containing text, images and buttons
<br>

#### `send` event structure
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* user identification value */
    
 "textContent": {}, /* used to send text */
    "imageContent": {}, /* used when sending a singular image */
    "compositeContent": {}, /* used when sending a composite message */
    
    "options": {
        "notification": true /* Used to send push */
    }
}
```

The `send` event can be used in the following cases:
* When sending an event from Talk Talk to the chatbot
* When sending an event in response to an event sent from Talk Talk to the chatbot
* When a chatbot sends an event to a specific event

The `send` event must be sent by selecting only one of `textContent`, `imageContent`, or `compositeContent`.<br>
The `notification` in `options` indicates whether a push should be sent.<br>
* `notification` can only be used in the third case of sending a `send` event.
* For `notification`, `default` is `false`. If you're not sending a push, you don't need to fill in the `options' field.
* Even if `notification` is sent as `true`, push cannot be sent if `user` is in the chat window, because messages can be received in real time.
* When a push is sent, a notification is sent to the 'Notification' of 'Naver Me' along with the message, and the push is delivered to the 'Naver App' installed on the mobile device.

Based on the above, it would be better to always send `notification` as `true`. However, `notification` can cause side effects in the following situations:
* If the 'user' switches the page to an external link provided by the chatbot while chatting with the chatbot in the TalkTalk chat window, they will receive a push because they have already left the TalkTalk page. However, we know that the 'user' is still in conversation with the chatbot. For example, you may receive a push while entering an address on a separate page.
* Whether a 'user' is in the chat window is determined by whether it is connected to the server through a 'Web Socket'. However, for fast page rendering, `Web Socket` uses `Lazy-Connection`, so if `notification` is sent as `true` at the same time as `open` event, `user` can receive push in chat window.

For the above reasons, it is recommended to use `default(false)`. The chatbot responds to the 'user' request in real time, so it is rare for the 'user' to send a response while leaving the chat window. However, if the above two cases do not apply and the chatbot needs to send an event according to a specific event, such as 'delivery departure' or 'reservation completed', it is recommended to use 'notification'.
<br>
<br>

### `echo` event
The `echo` event is an event that receives messages sent by agents to users and messages sent by chatbots to users in [Partner Center](https://partner.talk.naver.com/). The event name is `echo` and the userThe original event that we want to send to is stored in the `echoedEvent` property.<br>
To receive `echo` events, click **Developer Tools > API Settings > Select Event** and enable **echo**.
<br>

#### `echo` event structure

```json
{
  "event": "echo",
  "echoedEvent": "send",
  "user": "5KcCQTARWKNKv1IOvXwYQw",
  "partner": "wc8b1i",
  "textContent": {
    "text": "Your business card has been sent.",
    "inputType": "nameCard"
  },
  "options": {
    "mobile": false
  }
}
```

> **Note**
>
> The 'echo' event receives the message sent by the chatbot again, so if the message is retransmitted to the chatbot platform when the event occurs, the message is sent and received continuously Therefore, you need to accurately test those events and reflect them in your actual service.
<br>

### `action` event
The `action` event is an event that can be used when an additional action is required in the dialog. The `action` event can be sent using the `Webhook` response message or `send API`.<br>
The `action` information of the `action` event is as follows.
  * `typingOn`: The 'writing' image ( ![image](https://talk.naver.com/static/front/pc/img/typing.gif) ) is shown as a partner message. It lasts for 10 seconds if no further action is taken. If you send continuously, the retention time will be updated from the time of the new sending.
  * `typingOff`: The 'writing' image disappears from the dialog. Sending another message to the `send` event has the same effect.
```javascript
{
    "event": "action",
    "user": "al-2eGuGr5WQOnco1_V-FQ",
    "options": {
        "action": "typingOn" | "typingOff"
    }
}
```
> **Note**
>
> The `typingOn` and `typingOff` actions can be used to notify the user that a message is being processed if the message reception is delayed in the chat window. Or you can use it when you want to give the experience of talking slowly rather than responding to messages too quickly.
> In general, rather than using the `typingOff` action, send the message `send` event after sending `typingOn`.

<br>

### `persistentMenu` event

A `persistentMenu` (hereafter referred to as a fixed menu) is a menu that the user can access at all times during a conversation. It can be used to introduce chatbots to users, or to inform announcements and events.

![persistent_menu](/images/persistentMenu.png)

This menu should contain only functions that the user can launch at any time. For example, you can connect to a URL or receive code from the client and process it. Since it is a setting event, 'user identification value' is not required.

#### `persistentMenu` event structure
```json
{
"event":"persistentMenu",
"menuContent" : [{
"menus":
[{
"type":"TEXT",
"data":{
"title":"chatbot guide",
"code":"CHATBOT_GUIDE"
}
},{
"type":"LINK",
"data":{
"title":"event page",
"url":"http://your-pc-url.com/event",
"mobileUrl":"http://your-mobile-url.com/event"
}
},{
"type":"LINK",
"data":{
"title":"Call me",
"url":"tel:021234567"
}
},{
"type":"NESTED",
"data":{
"title":"Notice",
"menus":
[{
"type":"LINK",
"data":{
"title":"Exchange/Refund Information",
"url":"http://your-pc-url.com/guide",
"mobileUrl":"http://your-mobile-url.com/guide"
}
}]
}
}]
}]
}
```
<br>

#### `menuContent` List

`menuContent` is received in the form of a list. Send an empty list ([]) to initialize `menuContent`.

> **Note**
> * I left it as a `List` for development scalability. Currently it only accepts the first 'menuContent'.
> * If you send an empty List, the fixed menu is 'deleted'.
>
> ```json
>{
> "event":"persistentMenu",
> "menuContent" : []
>}
>```
<br>

#### `menuContent` Object

`menuContent` is a list containing various objects that can be put into the `persistentMenu` event. The object items that can be put are as follows.

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `menus` | Menu[] | Y | 'Menu' list. <br>- You can add up to four.<br>- `menus` cannot be `null`. |
<br>

#### `Menu` Object

The most basic form of a `Menu` object. Declare specific types in `type`, and put data appropriate for each type in `data`.

```
{
    "type": "menu type",
    "data": {
        /* MenuData */
    }
}
```

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `type` | string | Y | The type of the `menu` element. `TEXT`, `LINK`, `NESTED` |
| `data` | MenuData| Y | data of 'menu' element |
<br>

##### `MenuData` Object (`TEXT` type)

Similar to text type buttons. You can receive the value you want to identify in the `code` item and receive it from the chatbot client.

```json
{
    "type": "TEXT",
    "data": {
        "title": "title",
        "code": "CODE"
    }
}
```

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | Y | Text exposed in 'menu'. The 'text' that appears in the chat window when the 'user' clicks on the menu. <br>The maximum length is 20 characters. |
| `code` | string | Y | The 'code' sent to the client when the 'user' clicks on the menu. <br>The maximum length is 1,000 characters. |

<br>

##### `MenuData` Object (`LINK` type)

Similar to a link type button. The URL address is displayed in a new window according to the user's environment (PC, mobile).

```json
{
    "type": "LINK",
    "data": {
        "title": "title",
        "url": "http://your-pc-url.com",
        "mobileUrl": "http://your-mobile-url.com"
    }
}
```
| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | Y | Text exposed in 'menu'. The maximum length is 20 characters. |
| `url` | string | Y | The page URL to go to when you click ‘Menu’ in the PC chat window. <br>If you enter the value of `tel:{phone number}` in `url`, you can make a phone call on mobile. |
| `mobileUrl` | string | N | The page URL to go to when you click ‘Menu’ in the mobile chat window. <br>If there is a separate mobile URL, set ‘mobileUrl’ to connect to that URL on mobile. |

> **Note**
>
> * Opens links in 'current window' on mobile and opens links in 'new window' on PC.
> * On mobile, when returning to the chat window from the page moved to the 'current window', you must use 'back' or 'history.back()'.
> * Please refer to [Precautions when using LINK-type buttons] (#link-type-button-use-when-precautions-notes).
<br>


##### `MenuData` Object (`NESTED` type)
The `NESTED` type creates nested inner menus. Internal menus can be created up to 3 levels including the top menu.

```
{
    "type": "NESTED",
    "data": {
        "title": "title",
        "menus": [ /* Menu[] */ ]
    }
}
```

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | Y | Text exposed in 'menu'. The maximum length is 20 characters. |
| `menus` | Menu[] | Y | Submenu list. |
<br>

## Message type specification
<br>

### `textContent`

When the chatbot only wants to send a text message to the user, it only needs to pass data in the `text` field. The same goes for sending a message to the chatbot.format is used, in this case, the value entered by the chatbot is passed to the `code` item.
The `inputType` item contains information about which medium (or additional function) the text attribute was created with when the user clicks the add-on of Talk Talk during conversation with the chatbot.


```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* user identification value */
    
    "textContent": {
        "text": "Hello? This is Domino's Pizza ordering chatbot. Order 6 popular menu items quickly!", /* Text to be displayed in the chat window */
        "code": "", /* The code value received when clicking the button in compositeContent, code is not exposed in chat. */
        "inputType": "typing|button|sticker|vphone" /* This property is received only when receiving. It is a value indicating what medium the user inputs to the bot. */
    }
}
```

* Enter the text you want to send in `text`.
* Insert `\n` when a line break is needed.
* If a phone number is entered in the text, such as '010-1234-1234' or '01012341234', 'telto:' is automatically inserted when exposed in the chat window, so you can make a 'call' directly from your mobile device.
* If a URL is inserted in the text, such as `https://talk.naver.com/`, it is automatically converted to Anchor Tag (`<a>`) and exposed.
* The maximum text length must be within 10,000 characters, regardless of English/Korean.
* `code` can be used to check which button is pressed when the button built into `compositeContent` is clicked.
* When sending ‘compositeContent’, if you put ‘A’ and ‘B’ codes in each of the two buttons, you can check which button was pressed by checking the ‘code’ value when the user clicks the button.<br>

* The information of the `inputType` property value is as follows.
  * typing: This is when the user directly writes in the input box and sends it by pressing the ‘Send’ button.
  * button: This is when you respond by directly pressing various types of buttons sent by the bot. The `code` value is included as well.
  * sticker: In case of sending by clicking the sticker. You can respond when the user taps on a sticker.
  * vphone: This is the case when the ‘Request a consultation with a secure phone number’ button located on the right side of the sticker button is pressed. A safety phone number and the validity period of the phone number (in the form of yyyy-MM-dd) are provided.
    * Example: {"text":"050719003814,2017-11-03", "inputType":"vphone"}
  * product: This is when you start chatting with ‘Talk Talk Inquiry’ on the product page of the Smart Store/Shopping window or execute ‘Inquiry about other products’ from the Talk Talk mobile page widget The options property contains product information. When entering 'Tok Tok Inquiry', this textContent is transmitted together with the user's additional action such as typing.
  
  [Example of event in case of `inputType`: `product`]
```javascript
{
"event": "send",
"user": "al-2eGuGr5WQOnco1_V-FQ",
"textContent": {
"text": "Inquire about this product.",
"inputType": "product"
},
"options": {
"product": {
"name": "[Used] Android 4.0 learning with 200 step-by-step examples",
"url": "http://smartstore.naver.com/inflow/outlink/product?p=309672359&tr=tsf&site_preference=device",
"mobileUrl": "http://m.smartstore.naver.com/inflow/outlink/product?p=309672359&tr=tsf&site_preference=device",
"thumbUrl": "https://shop-phinf.pstatic.net/20150716_65/pqbdo_1437051783074LGw89_JPEG/43672814967807467_-164086553.jpg?type=f344",
"currencyPrice": "19,900 won",
"currencyMobilePrice": "19,900 won"
},
"mobile": false
}
}
```
<br>

### `imageContent`

`imageContent` is a speech bubble composed only of images. Since the size of the image is fixed, it is good for the user experience to send an image that has been tested and optimized.

```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* user identification value */
    
    "imageContent": {
        "imageUrl": "http://blogfiles5.naver.net/20130918_119/city0080_137946683395507ioT_JPEG/6.jpg" /* URL of image to send */
    }
}
```
* `imageUrl` must be an externally accessible URL. If the URL is accessible only from within, the transmission will fail.
* When the 'Talk Talk Server' accesses the given URL and downloads an image, the 'Content-Type' value of the 'HTTP' response header must match the image type.
* When ‘imageContent’ is sent to Talk Talk (1), the ‘Talk Talk Server’ accesses the URL once and downloads the image. (2) Downloaded images are stored in Talk Talk and exposed to ‘users’ as image URLs for Talk Talk.
* If you send `imageContent` to Toktok and receive a success response, you no longer need to maintain the image URL.
* Restrictions on images used for `imageContent` are as follows.
   * Images can be up to 20MB in size.
   * Available image formats are JPG, JPEG, PNG and GIF.
   * Both 'transparent background' and 'moving image' can be expressed in the chat window.
* If a problem occurs when sending `imageContent` to Tok Tok, please refer to the "imageContent error code table" for image processing.
* If an error occurs while sending `imageContent`, please refer to the "imageContent Error Code Table".
<br>

> **Tips to improve performance**
>
> The image URL specified in `imageUrl` is used to download an image from `Toktok` whenever a message is sent. You can speed up message delivery by pre-uploading frequently used images using the [Image Upload API] (/imageupload_api_v1.md).

<br>

### `compositeContent`

‘compositeContent’ is a message that can use multiple types of ‘components’ in a composite way.

![composite_message](/images/composite_message.jpg)

One 'composite' can contain the 'components' below.
  * `image`: an `image` element that exposes a single image
  * `elementList`: a `list` element that can be composed of `title`+`description1`+`description2`+`button`+`thumbnail`.
  * `title`: A slightly bold `title` element
  * `description`: a `description` element that is more blurred than `title`
  * `buttonList`: a list of `button` elements with various functions

You can compose a 'Carousel' by putting more than one 'composite' in a 'compositeContent'.<br>
You cannot change the exposure order of 'components'.
<br>

[use examples including all 'components']
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* user identification value */
    
    "compositeContent": {
        "compositeList":[
            {
                "title": "title",
                "description": "description",
                "image": {
                    "imageUrl": "http://shop1.phinf.naver.net/20170216_20/talktalk_14872437839327BN4b_PNG/menu_01.png"
                },
                "elementList": {
                    "type": "LIST",
                    "data": [
                        {
                            "title" : "list element title",
                            "description" : "List element description1",
                            "subDescription" : "List element description2",
                            "image" : {
                                "imageUrl": "http://shop1.phinf.naver.net/20170216_20/talktalk_14872437839327BN4b_PNG/menu_01.png"
                            },
                            "button" : {
                                "type": "TEXT", /* Only TEXT and LINK types are allowed for the button of the list element */
                                "data" : {
                                    "title": "element button", /* button name of list element (up to 4 characters) */
                                    "code" : "code"
                                }
                            }
                        }
                    ]
                },
                "buttonList": [
                    {
                        "type": "TEXT", /* Text type button - when the user clicks, the text of the button is sent*/
                        "data" : {
                            "title": "Text-type button", /* Button name exposed on the button (up to 18 characters)*/
                            "code" : "code" /* If code is defined, the code is inserted into the send event textContent sent by the user and transmitted (up to 1,000 characters)*/
                        }
                    },
                    {
                        "type": "LINK", /* Linked button - opens the page when the user clicks */
                        "data": {
                            "title": "Link-type button", /* Button name exposed on the button (up to 18 characters)*/
                            "url": "https://dominos-bot.talk.naver.com/view/menu/1", /* Link URL in Talk Talk PC version chat window */
                            "mobileUrl": "https://dominos-bot.talk.naver.com/view/menu/1#nafullscreen" /* Link URL in the chat window of the chattok mobile version */
                        }
                    },
                    {
                        "type": "OPTION", /* Optional button - 2depth button. When the user clicks, the button list is exposed at the bottom of the chat window. */
                        "data": {
                            "title" : "optional button",
                            "buttonList" :[
                                {
                                    "type": "TEXT", /* Only TEXT, LINK, and PAY types are allowed for optional buttons */
                                    "data" : {
                                        "title": "Option-Text Button", /* Button name exposed on optional button (up to 10 characters) */
                                        "code" : "code"
                                    }
                                },
                                {
                                    "type": "LINK",
                                    "data" : {
                                        "title": "option-link button",
                                        "url": "https://dominos-bot.talk.naver.com/view/menu/1",
                                        "mobileUrl": "https://dominos-bot.talk.naver.com/view/menu/1#nafullscreen"
                                    }
                                }
                            ]
                        }
                    },
                    {
                        "type": "PAY", /* Naver Pay payment button */
                        "data": {
                            "payKey": "wc8bls20170718002252151YjE1NzQwMD" /* Pay key */
                        }
                    }
                ]
            }
        ]
    }
}
```
<br>

#### `CompositeContent` Object

`CompositeContent` is expressed by grouping several elements constituting the speech bubble. Currently, only one Composite is supported, and it is received in the form of a list.

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `compositeList` | Composite[] | Y |Composite of list elements. <br>- You can add up to 10 `Composites`.<br>- `Composite` cannot be `null`. |

<br>

#### `Composite` Object

`Composite` is a speech bubble that can be created by combining several items. Please note the following:
* At least one of `title`, `description`, and `elementList` must exist in one `composite`.
* At least two properties of `title`, `description`, `elementList`, `image`, and `buttonList` must exist in one `composite`.

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | N | The value of the `title' element. <br>- The maximum length is 200 characters. <br>- Insert a `\n` if a line break is required. |
| `description` | string | N | 'Description' element value. <br>- The maximum length is 1000 characters. <br>- Insert a `\n` if a line break is required.|
| `image` | Image | N | 'image' element |
| `elementList` | ElementList | N | `element list` |
| `buttonList` | Button[] | N | A list of `button` elements. <br>- You can put up to 10 'buttons'.<br>- 'Button' cannot be 'null'. |

<br>

#### `Image` Object

See how to use [imageContent](#imagecontent) and tips for improving performance.

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `imageUrl` | String | Y | Image URL |

> **Note**
>
> * Recommended image size is 530px wide and 290px high. (Ratio 1.82:1)
> * Supported image formats are JPG, JPEG, PNG and GIF.

<br>

#### `ElementList` Object

 It is one of several elements of a Composite, and each Composite occupies a small area. It can receive multiple elements in the form of a list.

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `type` | string | Y | The type of list element. Currently, only the `LIST` type exists. |
| `data` | ElementData[] | Y | The data of the list element. <br>- You can put up to 3 'ElementData'. <br>- `ElementData` cannot be `null`. |

<br>

#### `ElementData` Object (`LIST` type)

'ElementData' is a small area in the Composite that sits between the image and the title.
It is recommended that this area be composed of brief contents that support Composite rather than complex contents.


| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | Y | 'Title'. <br>- You can enter up to 100 characters. <br>- Based on mobile (resolution 375, iPhone 6s), up to 10 characters are exposed, and more are indicated by ellipses (...). <br>- `title` is displayed as one line. |
| `description` | string | N | `Description 1`. <br>- You can enter up to 100 characters. <br>- Mobile (Resolution375, iPhone 6s), up to a maximum of 25 characters are exposed, and more are indicated by an ellipsis (...). <br>- Insert a `\n` if a line break is required. <br>- `description` is exposed with a maximum of 2 lines. |
| `subDescription` | string | N | `Description 2`. <br>- You can enter up to 100 characters. <br>- Based on mobile (resolution 375, iPhone 6s), up to 13 characters are exposed, and more than that is indicated by an ellipsis (...). <br>- `subDescription` is exposed as one line. |
| `image` | Image | N | 'image'. If no input is made, the default image is exposed. |
| `button` | Button | N | 'Button'. Only `TEXT` and `LINK` types are allowed and the length of `title` is limited to 10 characters. |

<br>

#### `Button` Object

The basic structure for presenting a button to the user is as follows. Data that can be put in for each button type is different, so be careful when configuring it.

```javascript
{
    "type": "button type",
    "data": {
        /* data */
    }
}
```

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `type` | string | Y | The type of the `button` element. `TEXT`, `LINK`, `OPTION`, `PAY` |
| `data` | ButtonData | Y | data of `button` element |
<br>

##### `ButtonData` Object (`TEXT` type)

Text and code created by the chatbot when configuring the button. Text is a user-visible value, and code is a value expressed only as an HTML attribute.
Since text and code are transmitted together, it is recommended to interpret the code value and proceed with the desired scenario.


```javascript
{
    "type": "TEXT",
    "data": {
        "title": "title",
        "code": "code"
    }
}
```

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | Y | Text exposed on 'button'. <br>- This is the 'text' sent when the 'user' clicks the button. <br>- The maximum length is 18 characters. |
| `code` | string | N | The 'code' sent when the 'user' clicks the button. The maximum length is 1,000 characters. |

<br>

##### `ButtonData` Object (`LINK` type)

If you click the link type button, a new mobile URL or URL page is displayed according to the user's environment. It is recommended to provide both the mobile version and the PC version on the page to be provided according to the user's environment.

```javascript
{
    "type": "LINK",
    "data": {
        "title": "text exposed on button",
        "url": "http://your-pc-url.com",
        "mobileUrl": "http://your-mobile-url.com"
    }
}
```

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | Y | Text exposed on 'button'. The maximum length is 18 characters. |
| `url` | string | Y | Page URL to go to when you click 'button' in the PC version chat window |
| `mobileUrl` | string | Y | Page URL to go to when you click 'Button' in the mobile version chat window |

> **Note**
>
> * Opens links in 'current window' on mobile and opens links in 'new window' on PC.
> * On mobile, when returning to the chat window from the page moved to the 'current window', you must use 'back' or 'history.back()'.
<br>

##### `ButtonData` Object (`OPTION` type)

If you click the 'OPTION' type button, the defined 'quick button' is displayed at the bottom. It disappears when the user clicks on an area outside the button area.

```javascript
{
    "type": "OPTION",
    "data": {
        "title": "text exposed on button",
        "buttonList": [
            {
                "type": "title",
                "data": {
                    /* button data */
                }
            }
        ]
    }
}
```

| key | type | Required | Description |
|:---:|:----:|:----:|------|
| `title` | string | Y | Text exposed on 'button'. The maximum length is 18 characters. |
| `buttonList` | Button[] | Y | A list of 'buttons' exposed at the bottom of the chat window when you click on 'buttons'. <br>- Up to 10 'buttons' can be inserted. <br>- 'button' cannot be 'null'. <br>- Only `TEXT`, `LINK`, and `PAY` types are allowed for `button`. <br>- `title` is limited to 10 characters. |

<br>

#### Notes on Using Composite Messages

* The types of `button` added to `buttonList` include `TEXT`, `LINK`, `OPTION`, and `PAY`.
* When using the `TEXT` type, the message that the user writes in the chat window and the value of the `text` property exposed on the button are delivered in the same format.
   * Usually by analyzing the 'text', you can figure out which button the 'user' pressed and proceed with the next process. However, if 'text' is difficult to parse, or if your chatbot is 'Stateless' and can't manage state, you can use 'code'. For example, when asking the user for 'gender' and 'age' in turn, if the 'user' selects a man ('code=1'), here's the 'age' (10s, 20s, 30s. ..) to make the code `1-10`, `1-20`, `1-30`... Then, when the user selects their 30s, a code value of '1-30' is passed to the chatbot, so that the 'user' is a male and 30s.
   * If the bot is `Stateful`, it is recommended to analyze `text` directly. That's because you can see all the text the user has pressed on a button or typed directly in 'compositeContent'.
* If you use the `LINK` type, you can use an external web page toktok.
   * The Talk Talk chat window provides both 'PC version' and 'Mobile version'. Even ‘user’ can talk with chatbot in ‘mobile version’ and continue the conversation in ‘PC version’. So for `LINK` type, `PC`/`mobile` URL must be set separately.

<br>
<br>

##### Precautions when using the LINK type button

Clicking the 'LINK' type button in the 'mobile' chat window opens a link in the 'current window'. You will be directed to an 'external web page' in the 'Talk Talk Chat Window'. When returning to the 'Talk Talk Chat Window' after moving to an 'external web page', do not use 'Page Move' **You must use `history.back()'**.<br>
When you 'move page' from 'external webpage' to 'talk chat window', if you press 'history.back()' in 'talk talk chat window', it goes back to 'external web page' instead of going to 'chat list' will go Normally, there is no problem if you go to the previous page by pressing the button in the ‘Talk Talk Chat Window’, but if you press the Android physical button to ‘history.back()’, you cannot be guided to the ‘Chat List’.<br>
In order to return to the 'Talk Talk chat window' with 'history.back()' from the 'external web page', the 'external web page' must not be moved to multiple pages or must be made in the SPA (Single-Page Application) method. Otherwise, make it into multiple pages, but even the 'external webpage' should be driven back to the previous page only with a thorough 'history.back()'.
<br>
<br>

##### Creating external web pages using tokens
Complex form data that is difficult to receive interactively from 'users' can be received through a separate 'external web page'. In this case, the question arises how to identify 'partner' and 'user' on 'external webpage'. You can pass both 'partner' and 'user' identification values ​​as Get parameters, but you must keep 'private' as the 'user' identification value is simple enough to match arbitrarily.
So, you need to use a token (`token`) to prevent the `user` identification value from being exposed to the outside. When sending `compositeContent` via Talk Talk, the token is passed as a parameter to the URL of the `external web page` inserted.
* When sending `compositeContent` to TalkTalk, a random character corresponding to the already known `user` identification value is stored as `key`, the `user` identification value is stored as `value`, and `key` is used as the token value for URL add it as a parameter.
* When an 'external webpage' is called, you can use the token as a 'key' in the 'Server-Side' to obtain a user identification value.<br>

If the token method is implemented as above, even if the same user is the same, the tokens put into the `compositeContent` will change every time, and if you use a token that is arbitrarily and unpredictably long, you can more safely protect the user identification value.
<br>
<br>

### quick button

The quick button is located at the bottom of the chat screen.So if you send other content under the quick button it will disappear. It also disappears after clicking, so you can only click once.

![btn_quick](/images/btn_quick.jpg)

* A function similar to the `OPTION` type button of `compositeContent` is also provided for `textContent` and `imageContent`.
* When you send a message, a 'button' is listed below the chat window.
* You can use `TEXT`, `LINK`, and `PAY` type buttons.
* The number of characters for the button ‘title’ is limited to 10 characters.
<br>

#### Apply a quick button to `textContent`
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* user identification value */
    
    "textContent": {
        "text": "text",
        "code": "code",
        "quickReply": { /* quick button - quick reply */
            "buttonList": [ /* button list */
                {
                    "type": "TEXT", /* TEXT, LINK, PAY type buttons are allowed. */
                    "data": {
                        /* button data */
                    }
                }
            ]
        }
    }
}
```
<br>

#### Apply a quick button to `imageContent`
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* user identification value */
    
    "imageContent": {
        "imageUrl": "http://blogfiles5.naver.net/20130918_119/city0080_137946683395507ioT_JPEG/6.jpg", /* URL of image to send */
        "quickReply": { /* quick button - quick reply */
            "buttonList": [ /* button list */
                {
                    "type": "TEXT",
                    "data": {
                        /* button data */
                    }
                }
            ]
        }
    }
}
```
<br>

#### Apply quick button to `compositeContent`
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* user identification value */
    
    "compositeContent": {
        "compositeList": [
            /* composite data */
        ],
        "quickReply": {
            "buttonList": [
                {
                    "type": "TEXT",
                    "data": {
                        /* button data */
                    }
                }
            ]
        }
    }
}
```
<br>

## error specification

### error example
```javascript
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "success": false,
  "resultCode": "02",
  "resultMessage": "Error parsing request json string"
}
```
<br>

### error structure

| key | name | type | Required | Description |
|:---:|:----:|:----:|:----:|------|
| `success` | Success or not | boolean | Y | Whether the API call was successful. In case of failure, it responds according to the resultCode. |
| `resultCode` | result code | string | Y | Code "00" is success, otherwise it returns the cause code value in case of failure. |
| `resultMessage` | Code Description | string | N | The resultCode is specifically described. |
<br>

### Common Error Code Table

| success | resultCode | resultMessage | Description |
|:-------:|:----------:|---------------|------|
| true | `00` | success | Success |
| false | `01` | Authorization information error | Authorization information is incorrect or you used information that has already expired |
| false | `02` | request json string parsing error | If there is a problem with the JSON structure or missing a required value |
| false | `99` | {specific details of failure} | Detailed explanation of various errors that may occur during processing other than 01 and 02 |

<br>

### imageContent error code table

| success | resultCode | resultMessage | Description |
|:-------:|:----------:|---------------|------|
| false | `IMG-01` | Image Upload - Format Mismatch or Processing Failed | If the format of the image you are uploading is not JPG, JPEG, PNG, GIF, or otherwise |
| false | `IMG-02` | Image Upload - Transmission/Processing Timeout | If the download time of the image you upload exceeds 10 seconds |
| false | `IMG-03` | Upload Image - Exceeded Size | If the image you upload is over 20 MB |
<br>
<br>
