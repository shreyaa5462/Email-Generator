# AI Email Assistant

**Chrome Extensions -**\
**Gemini API -**

If your application want to communicate with Googe's Gemini AI model
then we make use of gemini api

You generate api key - a secret key which you send along with your
request

Application ---------------------------------------------------------\>.
Gemini AI model

                                secret key + request
                                (api key)

MutationObserver

``` bash
curl "https://generativelanguage.googleapis.com/v1beta/models/gemini-flash-latest:generateContent" \
  -H 'Content-Type: application/json' \
  -H 'X-goog-api-key: YOUR_GEMINI_API_KEY' \
  -X POST \
  -d '{
    "contents": [
      {
        "parts": [
          {
            "text": "Explain how AI works in a few words"
          }
        ]
      }
    ]
  }'
```

**2. How does our project work?**

Step 1: Receive an email

User receives an email in Gmail.

Step 2: Gmail Extension

User clicks the Reply with AI button.

Step 3: Select Tone

User selects a preferred tone, such as Professional, Friendly or Casual.

Step 4: Backend Generate API

Receives original email content and selected tone.

Step 5: Gemini API

Backend sends the prompt to Gemini using the Gemini API key.

Step 6: Generated Reply

AI-generated response is returned and displayed in Gmail's reply editor.

The controller handles incoming HTTP requests from the Gmail extension
and forwards them to the service layer.

Gmail extension sends a POST request.

@RequestBody converts JSON into EmailRequest.

``` java
public class EmailRequest {

    private String emailContent;

    private String tone;

}
```

Controller calls generateEmailReply() from the service layer.

Service generates the AI email reply.

ResponseEntity.ok() returns the response with HTTP status 200 OK.

API Endpoint

POST /api/email/generate

@CrossOrigin(origins = "\*")

@CrossOrigin → Enables cross-origin access.

It tells the backend to allow requests coming from any website or
origin.

Our Gmail Chrome Extension sends an API request to our Spring Boot
backend.

Web client

We use WebClient to communicate with the Gemini API through HTTP
requests.

build() creates the WebClient instance.

**Injecting Gemini configuration**

``` java
@Value("${gemini.api.url}")

private String geminiApiUrl;

@Value("${gemini.api.key}")

private String geminiApiKey;
```

-   gemini.api.url: Gemini API endpoint.
-   gemini.api.key: Authentication key used to access Gemini API.

Both values are stored in application.properties and injected using
@Value.

Web Flux -

Its main purpose is to build applications that handle requests using
non-blocking, asynchronous communication.

WebClient comes from Spring WebFlux and helps us communicate with Gemini
API.

Important: We are using .block() in our code, so our current Gemini API
call is synchronous, even though WebClient supports non-blocking
communication.

WebClient comes from Spring WebFlux and helps us communicate with Gemini
API.

Important: We are using .block() in our code, so our current Gemini API
call is synchronous, even though WebClient supports non-blocking
communication.

generateEmailReply()

This method performs the complete email generation process.

Step 1: Build the prompt

``` java
String prompt = buildPrompt(emailRequest);
```

Creates a prompt using the original email content and selected tone.

Converts our prompt into the JSON structure expected by Gemini API. -
crafting of prompt

Send HTTP request to gemini endpoint

``` java
String response = webClient.post()

    .uri(geminiApiUrl + geminiApiKey)

    .header("Content-Type", "application/json")

    .bodyValue(requestBody)

    .retrieve()

    .bodyToMono(String.class)

    .block();
```

**5. extractResponseContent()**

``` java
ObjectMapper mapper = new ObjectMapper();

JsonNode rootNode = mapper.readTree(response);
```

Gemini returns a JSON response.

We use Jackson's ObjectMapper to parse that JSON and extract the
generated email text.

``` text
candidates
    └── content
          └── parts
                └── text
```

**buildPrompt()**

This method creates the prompt that we send to Gemini.

**private String buildPrompt(EmailRequest emailRequest)**

It takes two important inputs:

1.  Original email content.
2.  Selected tone.

For example:

Input

-   Email: Please share the project report by tomorrow.
-   Tone: Professional

Generated prompt

**Generate a professional email reply**

**for the following email content.**

**Please don't generate a subject line.**

**Use a Professional tone.**

**Original email:**

**Please share the project report by tomorrow.**

``` text
Controller
    |
    v
generateEmailReply()
    |
    v
buildPrompt()
    |
    v
Prepare JSON Request
    |
    v
WebClient POST Request
    |
    v
Gemini API
    |
    v
Receive JSON Response
    |
    v
extractResponseContent()
    |
    v
Return Generated Email
```

Making chrome extensions - it adds extra functionality and to your
chrome example is - brainly , google translator

manifest.json is the main configuration file of a Chrome Extension. It
tells Chrome what our extension is, what permissions it needs, and which
scripts it should execute.

``` json
{
  "name": "Email Writer Assitant",
  "description": "AI-powered email reply generator",
  "version": "1.0",
  "manifest_version": 3
}
```

-   name: Name of our extension.

-   description: Purpose of our extension.

-   version: Current extension version.

-   manifest_version: Chrome extension platform version. We are using
    Manifest V3.

-   "permissions": \["activeTab", "storage"\]

-   activeTab: Provides temporary access to the currently active tab
    after a user action. storage: Allows our extension to store and
    retrieve data using Chrome storage APIs.

``` json
"host_permissions": [
    "http://localhost:8080/*",
    "*://mail.google.com/*"
]
```

Defines which websites our extension needs access to.

-   localhost:8080: Our Spring Boot backend.
-   mail.google.com: Gmail website.
-   \*://: Allows HTTP or HTTPS.

``` json
"content_scripts": [
    {
        "js": ["content.js"],
        "matches": ["*://mail.google.com/*"],
        "css": ["content.css"],
        "run_at": "document_end"
    }
]
```

This is particularly important for our project.

-   js: JavaScript file injected into Gmail.
-   matches: Specifies where the script should execute.
-   css: CSS file used for styling our extension elements.
-   run_at: Specifies when the content script executes.

document_end means the script runs after the DOM has been loaded.

**Web Accessible Resources**

``` json
"web_accessible_resources": [
    {
        "resources": ["icons/*"],
        "matches": ["*://mail.google.com/*"]
    }
]
```

Allows Gmail pages to access the specified extension resources, such as
our icons.

-   resources: Files we want to expose.
-   matches: Websites allowed to access those resources.

``` json
"action": {
    "default_title": "Email Writer Assitant"
}
```

Defines the extension's toolbar action and its tooltip when the user
hovers over its icon.

**Script-**

1.  createAIButton() -- Gmail mein AI Reply button create karta hai.
2.  getEmailContent() -- Original email ka content extract karta hai.
3.  findComposeToolbar() -- Gmail ke compose toolbar ko find karta hai.
4.  injectButton() -- Toolbar mein AI Reply button inject karta hai.
5.  Button click event -- Button click hone par:
    -   Email content extract hota hai.
    -   Backend API ko POST request bheji jaati hai.
    -   Email content aur tone (professional) request body mein send
        hote hain.
    -   Backend se generated reply receive hota hai.
    -   Reply Gmail ke compose box mein insert ho jaata hai.
6.  
7.  MutationObserver -- Gmail ke DOM mein changes observe karta hai aur
    compose window detect hone par AI Reply button inject karwata hai.
