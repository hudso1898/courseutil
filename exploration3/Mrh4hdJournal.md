# Matt Hudson - Exploration 3 Journal

**Full Stack Application - MEAN stack w/ Angular Material**

## Code

The frontend is deployed on my web server: https://www.hudso.dev/exploration3

The backend API is running on port **9080** on my server (port *8080* is still reserved for my Exploration 2 Node Server) - https://www.hudso.dev:9080. See the
rest of the journal for the specific accessible endpoints.

You can access the front-end through a web browser on your computer *and* your phone! I designed the UX to look good on both desktop computers and mobile devices. It's not as
powerful or sleek as Ionic, obviously, but it's a good start.

## What frameworks did you choose and why?

For this exploration, I decided to build off/integrate both of my previous explorations to form a complete MEAN stack
application. Since this is an industry-standard way of making a full-stack application, I figured this would be the most
beneficial to me. I wanted to build off what I still wanted to learn about those frameworks; specifically, integrating them
together by 1) Building an API that can serve data on demand using Node/Express/MongoDB, and 2) Accessing an API through
Angular and using that data in the front end. I also wanted to expand my knowledge of front-end frameworks, specifically Angular Material. I used Material in
Challenge 4; however, I didn't write a whole journal on Material, and there are plenty of other Material elements to explore.

As for the code I wrote, I decided (*per Wergeles' recommendation*) to expand Challenge 4 in the following ways to demonstrate my exploration of building
full-stack MEAN applications:
    
    1) Linking the Angular front-end to a back-end Node/Express/Mongo service to store the data previously held in LocalStorage. 
    In this way, I can explore and demonstrate linking the front/back ends together. This also persists the data, solving the lasting
    problem of Challenge 4: the data is only local to the browser. Persisting data is essential in the real world!
    2) Implementing new features in WergelEats; specifically, having user accounts and creating new restrictions/redesigning/new
    components based on this feature. In this way, I can explore and demonstrate implementing a full-stack development feature
    from scratch (this is to ensure my exploration involves building code from scratch, since this feature does not exist in
    any form, currently).
    3) Exploring and defining Material components I used in Challenge 4, as well as exploring, defining, and utilizing new Material
    components as necessary to improve the user experience.

## What did you learn about/accomplish in these frameworks?

### [0] Initializing the backend API

To start off, I needed to create the back-end API which my Angular application accesses. I already have a node server running on port 8080 for Exploration 2, so I needed to create a
separate server for this exploration. I could have used the same server; however, having separate servers allows a separation of concerns - so the new server deals only with Exploration 3
and vice versa. So, I started by reusing a lot of the index.js and package.json code - I stripped down index.js to its bare bones express - returning a simple JSON object at its base URL,
changing the listening port to **9080**, and I updated `package.json` to a different app name. Then, I allowed port `9080/tcp` on both the EC2 firewall (accessed through the EC2 console)
and `ufw` (accessed through SSH). I went back and forth from running the server manually (for debugging purposes), but I added the server to `pm2` in order to run in the background like my
original node server: `pm2 start index.js --name wergelNode`. At this point, I was able to access the API through https://www.hudso.dev:9080.

![api-alive](../screenshots/api-alive.png)

### [1] Reimplementing LocalStorage in the API
Once this was done, I defined two ways to access the city data:

- /get/cities [GET] => returns the array of cities as JSON (stringified)
- /set/cities [POST] (body.cities: Array<City>) => provides the updated array of cities to be set in the database

Like my Exploration 2, the server runs Node and uses `Express` to respond to particular queries, so i.e.

```javascript
app.get('/path/to/endpont', (req,res) => {
// business logic goes here
});
// similarly,
app.post('/path/to/endpoint', (req,res) => {
// business logic goes here
// access POST body parameters as follows:
let cityArray = req.body.cities;
// other logic....
}
```

In the `/get/cities` API, I use a `find({})` method to obtain the document. Since I didn't want to edit how I stored the data (i.e. everything is in one array of Cities, nested within itself), I
made a collection `cities` with one document with two properties: `name: 'cities', cities: <array of cities>`. So I use this method to return the array of cities as stringified JSON:
```javascript
dbo.collection('cities').find({}).toArray((err, results) => {
// results has one document: the cities document
for (let result of results) {
res.write(JSON.stringify(result.cities));
// other logic here
}
});
```

For the `/set/cities` API, I use either `replaceOne({name: 'cities'})` or `insertOne()` to update the database. This is why I need the `name` property - since I need to specify search criteria 
to obtain the singular document. I use insertOne() as a backup in case the document does not exist; once the database is being regularly accessed, replaceOne() will always be called. I use this
instead of `updateOne()` since Update specifies a changeset, whereas I need to replace the entire array.

I also utilized cloud-based MongoDB Atlas. I connect to the database using my credentials, then I can use the `MongoClient` interface to act on the database. Once this was complete, I
moved on to specifying the Angular:

### [2] Reimplementing LocalStorage in Angular => HttpClient and Observables

To start off, I looked at my getter and setter functions that access LocalStorage (i.e. `localStorage.set('cities', cities)` and such) and looked into how to do this with my API. I found
the HttpClient interface, which works like this:

Import the HttpClientModule in your AppModule, then import HttpClient in your DataService. Then, add this like you would a service or any other injectible:
```javascript
constructor(private http: HttpClient){}
```

After this, you can access the `this.http.get()` and `this.http.post()` methods. These methods send AJAX requests! The first parameter of both of these functions is the URL to
go to, so i.e. `www.hudso.dev:9080/get/cities`. For `http.post()`, there is a second parameter, which is the body of the POST request. This is the object that contains the specific data
you send along with the request, so i.e I need to specify an object containing my cities array here, in order to send this to my API.

However, it isn't just this simple, because the **A** in AJAX is tricky when it comes to Angular. These requests happen *asyncronously*, so these functions return an *Observable*. An
Observable is an object that represents something you can *observe*, like an AJAX request. The `subscribe(func: Function)` method is tacked onto an observable to define a
function to execute upon completion of the request, in this case an AJAX request. This function has a single parameter to represent the response body. So, you define an anonymous
inner function that will run when the AJAX request is done. 

An important note here: **you must use .subscribe() even if you don't do anything with the response; otherwise, the AJAX request will not run!** This is because Observables are
*lazy* - they only run when their result is *needed*. So if no processes are subscribed to the Observable, it has no need to run, and as such will not. 

I created a single method in my `DataService` that returns an observable of the 'get/cities' endpoint. From my components, I can subscribe to this observable and get that data when it
arrives. However, since this now occurs asyncronously, I needed to adjust my components, since previously it got its data syncronously from LocalStorage. So, I was getting blank views,
and they weren't updating when the AJAX response came back. Using the `getCityAPI()` method, I can act on this Observable - see my *problems* below for how I fixed this.

Another component I added at this point was loading icons. Since the AJAX request takes time (and presumably could take a *long* time if the Internet is slow, for example), I needed a way
to convey to the user that the page was loading. This was easily accomplished with Material's `<mat-spinner>` - essentially a loading icon. To use this, I have a property in each component
`loaded = false` that I use with `*ngIf` statements. So, I display the loading icon if the AJAX request is still going through - in the Observable's `subscribe()` method, I set `loaded = true`.
Once this happens, the `*ngIf` statements switch over, and the content shows instead of the loading icon. I also use this with my endpoints - if the URL is invalid (i.e. the city/restaurant) is
not found, loaded = true, but cityNotFound = true as well. Therefore, I can define to show the error dialog instead of the cards.

At this point, the app persisted data correctly, and I was ready to start implementing the users part of the exploration.

### [3] Implementing Basic User Accounts in the API

To begin developing the users component of the application, I first made a decision on how to represent the user accounts in MongoDB. I decided to have each user as a document; therefore,
I could utilize some of the more advanced features of Mongo and search through a collection. So I made a collection `users` inside of my `wergeleats` database. Each document has
this format (also added to my `types.ts`:
```javascript
export interface User {
id: String; // id generated by idService
username: String; // unique username, emphasis on unique
password: String; // *encrypted* password. I would never dare store passwords in plaintext!
name: String; // Full Name of the user. This does not need to be unique.
}
```
In my backend, I defined these endpoints to access the users API:

- /get/users [GET] => returns a JSON array containing the list of all *usernames*. No other information is returned, since this is sensitive information!
- /users/addUser [POST] (body.user: User) => adds the User to the system. The endpoint recieves a plaintext password, and encrypts it server-side before storing to MongoDB.
- /users/login [POST] (body.username: String, body.password: String) => provided the username and (plain) password, checks whether these credentials are correct (i.e. there exists a user
with username and the right password). Returns JSON: if invalid: `{valid: false}`, if valid: `{valid: true, user: (id, username, name)}`. This information is used to log the
user in.

In Node/Express, I used the `bcrypt` module to do password encryption. I decided to use the *syncronous* methods instead of *asyncronous* because of the importance of password validity.
I probably *could* have used the async methods, but I felt this was easier and didn't have a huge performance impact. I call the `bcrypt.hashSync(plaintextPassword, 10)` method to
generate the hashes. The `10` represents the salt parameter: bcrypt uses hash/salt, which adds another layer of security to passwords. Then, I can use the 
`bcrypt.compareSync(hash, plaintextPassword): boolean` method to compare in the login endpoint. So, I find documents where username == the provided username, then I compare
their passwords. If both of these are a match, the login is valid. Else, the login is invalid.

### [4] Implementing Basic User Accounts in Angular

To utilize this API, I created a new service `UserService` which handles access to the Users API. It has an object representing the current user (if any) that other components can access
using `get`. I then created two new components: `LoginComponent` and `AddUserComponent` that handle logging in and adding a new user to the system. Both of these components
use `MatDialog` to display a dialog like my previous forms. I created these forms and a few extra validators for the `AddUserComponent`. For example, I added a validator to check whether
the password and confirmPassword fields are equal (to confirm the password). I also created a validator to disallow duplicate user accounts being created by checking whether the provided
username is in the array of usernames. I get this array by calling the API `get/users` to set the `usernames` array in my `UserService`.

As to handle whether a user is logged in or not, I actually utilized a familiar friend: `LocalStorage`! So, whenever the login is successful, I set a value in localStorage containing the user info: 
`localStorage.setItem('loggedIn', user)`. So, if that item is present, it means the user is logged in, and vice versa. This is equivilient to how I used the `$[SESSION]` array in PHP.
Essentially, I am setting a session variable for a user being logged in; if it isn't present, there is no user logged in.

Lastly, I edited the `AppComponent` toolbar to change functionality depending on the login status. I do this with `*ngIf` statements while reading from the `UserService`'s `loggedIn: boolean`
attribute using `get`. If the user is not logged in, there are two functions: login and add user. If the user is logged in, there are four functions: the 3 add functions (city, restaurant, review) and
a new function: logout. The logout function is simple: it just calls `localStorage.removeItem('loggedIn')` in the `UserService` and removes the current user object set in the service. By
doing this, the view immediately updates to be back to not being logged in.

Another note, is that the login process is again asyncronous. Therefore, my `login()` method in `UserService` returns an `Observable`. So in my dialog component, I subscribe to that
observable and perform actions based on the result. If valid, the user is logged in, display a success message, and close the dialog. If not valid, display the message but **don't** close the dialog!
This way, the user can retry without having to open the dialog all over again.

### [5] Adding restrictions based on Login Status to Review

So at this stage, anyone can view the system (cities, restaurants, reviews) but one has to be logged in in order to *add* anything. Cities and restaurants don't have an associated user, but
reviews do. Therefore I had to redesign the `AddReviewComponent` to accomodate this. First, I removed the `name` field from the form. This is because the name is now grabbed from the
`UserService`. Since the user has to be logged in in order to add a review, this value is guaranteed to be there. I also grab the `username`, because this is the unique identifier of the
user. I made a modification to my `User` type to include a `authorUsername: String;` field. So now, each review also has the username of the user who added it. Then, I edited the validator
function to check the `username` of reviews rather than the author. Once this was done, no user can add a duplicate review for a single restaurant.

I also updated the `ReviewComponent` view to add the `authorUsername` in each `<mat-card>`.


### [6] Material Redesign => Material Tooltips

At this point, my exploration was nearly complete. The only issue I had was the toolbar - I wanted the application to look good on mobile devices. It would be a great deal of work to change the whole
project to Ionic, and I wanted to save that framework for Challenge 5. The problem was that the text next to each icon was overlapping and clustering on mobile devices. The solution I found was
to remove the static title text (previously in `<span>` tags) with Material tooltips! On mobile devices, these icons are sufficient, and are becoming of a mobile application. For a desktop application,
I still wanted to display the title of each component, and with Material Tooltips I can!

So, I added `matTooltip='message` to each of my buttons in the Material Toolbar. Now, when you hover over the icons, a tooltip is displayed containing the button's title! Furthermore, mobile
devices just show the icons now, and it fits perfectly! I'm very happy with this design.

### [7] Mini Material Encyclopedia

The last bit of this exploration covers my work both this week and last week, delving into Angular Material. Since Material wasn't *required* for Challenge4, I wanted to make a short encyclopedia
detailing my findings on Material and how I used them in my project. One aspect I discovered is that you have to import *every* Material Module to get a particular component to work. You do
this in the `AppModule`. 

**Material Cards**

Material Cards are like segments of a view. They can contain a header (which can contain a title, subtitle, and avatar (small circular image like a profile picture)), an image, content, and actions
(buttons). By default these cover the entire screen like a `<div>` element would, so I had to specify specific height/width values. Talking to Prof. Wergeles on Wednesday, I discovered there are also
different sizes of cards that are more adaptive. In my applicatoin, I use mat-cards extensively to represent cities/restaurants/reviews. So in each of my views, I have a card for each element, generated
using an `*ngFor` structural directive. I then display these in an `inline-grid` fashion. This makes the view beautiful, informative, and responsive!

Syntax:
```html
<mat-card *ngFor='let something of someArray'>
    <mat-card-header>
        <mat-card-title></mat-card-title>
        <mat-card-subtitle></mat-card-subtitle>
        <img mat-card-avatar src='some/file/path'>
    </mat-card-header>
    <img mat-card-image src='some/file/path'>
    <mat-card-content>
    <!-- some content goes here -->
    </mat-card-content>
    <mat-card-actions>
    <!-- buttons go here like View Restaurants, etc. -->
    </mat-card-actions>
</mat-card>
```

**Material Toolbar**

The toolbar is like the app header: it exists at the top of the page and contains items that involve navigation, actions, etc. It shows up as a colored bar and can contain various actions. You
can set the color of the element by specifiying the 'color' attribute (basic, primary, accent, warning, etc.). This goes for many other Material components like icons, text, etc. as well.
You can also specify many rows in the toolbar. I used this to add a row to display the username if the user is currently logged in.

Syntax:
```html
<mat-toolbar color=primary>
    <mat-toolbar-row>
    // actions here
    </mat-toolbar-row>
</mat-toolbar>
```

**Material Spinner**

This is a new component I used in this exploration. The material spinner is a predefined and **animated** loading icon. This is especially useful since my application is now connected to an external
API, and asyncronous AJAX requests can take some time to respond. Therefore, you can use the Material Spinner to convey that the page is loading. You can switch between the icon and content
by using `*ngIf` with a boolean property in the component's TS.

Also, you can style the spinner to show in the center of the view instead of the top-left corner (default). I did this by setting the spinner's `class=loading` with this css: `.loading { top: 50%; left: 50%; position: fixed;}`.

Syntax:
```html
<mat-spinner class=loading *ngIf='!loaded'></mat-spinner>
<div id=componentConent *ngIf='loaded'>
<!-- the content of the view i.e. cities goes here, only displayed if the content is loaded
</div>
```

**Material Forms (mat-form-field, mat-select, mat-option)**

`<mat-form-field>` tags are used to contain individual fields in a `<form<>`. These add custom styling and response to invalidation; I found these great to use with my dialogs. You specify
the form-field, then the `<input>` tag inside using `matInput`. One note is that this only works for some input values; for example, if you use a `<mat-slider>`, `<mat-slide-toggle>`, etc.
you don't need a `<mat-form-field>` tag. Really, this is useful with text input.

Syntax:
```html
<mat-form-field>
<input matInput etc>
</mat-form-field>
```

**Material Forms - Errors (mat-error)**

Within a `mat-form-field` or `mat-select`, you can specify a `<mat-error>` tag. This adds an error message whenever the form field is invalid. The huge benefit of this is that this shows
*automatically whenever the field is invalid!* So, there's no need to use `*ngIf` statements, since it's inherently built in. The message is specified by the inner html. I used interpolation with
either static messages (if there's only one invalid state possible) or a function (if there could be multiple ways the value is invalid): `<mat-error>{{getInvalidMessage()}}</mat-error>`.

**Material Slider/Slide Toggle**

The slider and slide toggle act as a spectrum slider (i.e from 0 to 10 for exampleP) and as a boolean toggle (yes/no, true/false, etc), respectively. The slide toggle is the easiest to use:
`<mat-slide-toggle></mat-slide-toggle>` is all you need, though to use it in a form, you need matInput, formControlName, etc. For the slider, you need to specify the `min`, `max`, `step`, and
`value` methods to set the range, what intervals your values are between (i.e. a step of `1` would increment the values as integers). You can also add the `thumbLabel` attribute to the tag to show
a small popup indicating the value: `<mat-slider thumbLabel min=1 max=5 step=1></mat-slider>`.

**Material Tooltip**

Like explained earlier, the tooltip is a small bit of dialog that displays when the user hovers over a specific element. This is a new component I used in this exploration to replace the bulky static
titles for my actions in my Material Toolbar. This is probably the easiest component to implement: simply specify `matTooltip='some message'` in the tag you want the tooltip to show over.

**Material Buttons/Links**

Material buttons are handy and beautiful, since they have this nice ripple effect by default. All one needs to do to turn a button into a Material Button is to add `mat-button`  to a `<button>` tag.
You can also specify the `color` attribute of the button like the toolbar. This also applies to `<a>` tags as well. You should use an `<a>` tag to change views, i.e. change components such as
viewing restaurants of a city when clicked. You should use a `<button>` to do some actions, such as accessing a dialog like adding a review.

**Material Dialog**

Material Dialogs are awesome to handle forms! It works by loading a *separate* component into the dialog. It greys out the backgorund, and closes when the user clicks out of the dialog. I used
this component extensively when dealing with my reactive forms, but there are a few kinks to implementing it. First, you inject the dialog component into whatever component you are calling
dialogs from:
```javascript
constructor(private dialog: MatDialog){}
```
Then, you can call the `open(component: Component)` on this dialog reference to open the dialog with the specified component. One problem is that you have to specify the dialog
components (i.e. `AddReviewComponent`) in your `AppModule`'s `entryComponents: Array[Component]`. Otherwise, the dialog will not load correctly.

In the dialog components themselves, you inject a reference to the current dialog. You need to specify a type, which is the current component:
```javascirpt
constructor(private dialogRef: MatDialogRef<ThisComponent>){}
```

You can then call `this.dialogRef.close()` to close the dialog. Within the component, you can perform all sorts of logic, form controls, etc. 

**Material Snackbar**

The snackbar is a small, non-intrustive notification system. Whereas alerts are annoying, the snack bar pops up at the bottom of the browser. I use this extensively to convey feedback to
the user whenever they add content, try to login (successful or otherwise), etc. The syntax is fairly simply: inject the snackbar in your component first:
```javascript
constructor(private snackBar: MatSnackBar){}
```

Then, you can call this method to show a notification: `this.snackBar.open(message: String, null, {duration: <some number>});`. You specify the duration in miliseconds for
how long the notification should be displayed. 

**Material Icons**

Lastly, material icons provide a standard set of icons to use in your application. These look beautiful, sleek, and modern, and are extremely easy to use. All you need to do is specify a
`<mat-icon>` tag, with the inner value being the name of the icon you want to import. So for example, `<mat-icon>people</mat-icon>` will display a people icon. There's a list of
these icons here: https://material.io/resources/icons/

## What do you still want to learn about these frameworks?

One aspect of developing this application I noticed is that maintaining data integrity is difficult. Since the data needs to be persistent, the database plays a crucial role in the application; however,
the more users that use this site, the more likely conflicting data could arise. For example, what happens if two users submit conflicting city arrays at the same time? Data could be lost, misconstrued, 
etc. This isn't a problem in a small application like this; but it's very important in an industrial application. Therefore, I would look into how to maintain data integrity with Mongo. I think the best
bet for this application would be to change how the data is stored in MongoDB (removing the nested objects and dividing the information into separate documents for each city, restaurant, etc).
This is out of scope for this exploration, but would make a great improvement.

I also would look into performance scalability; my intuition is that the login service might be slow since the compare function is syncronous. However, this may not be, since Node can handle requests
asyncronously. However, it would be interesting to see the metrics of a widely-utilized site, and how developers improve performance for large-scale applications.

Lastly, a quick practical improvement would be to add an UpdateReview component. Since a user can review a restaurant only once, they may change their mind or want to delete their review. So,
the logical next steps for this project would to be to add update and delete capability. I imagine this would be a quick fix, and would complete the whole CRUD interface of the application. Another
small improvement would be to sort the view according to some value. For example, the cities/restaurants/reviews are displayed in the order in which they were *added*. An improvement would
be to sort them based on some value (i.e. sort cities by state, then by name), etc.

## What problems did you run into?

The first problem I encountered was that my API was unable to return any data to my application. It worked fine through the browser (i.e. https://www.hudso.dev:9080/get/cities worked fne)
but it wouldn't work when the http request was made in Angular. Looking at the console, I realized it was a CORS error; the request was being denied since it was a different origin! So,
putting my CORS knowledge to the test (and looking up some handy documentation) I added these lines into my `index.js` file to allow cross-origin requests:

```javascript
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});
```
This allows cross-origin-requests to any other origin, thereby allowing the web browser to make an AJAX request for JSON. In fact, once this was in place, you can see CORS in action by
inspecting the `network` part of the browser. Here, the browser is making a *preflight request* when getting/posting data.

![cors](../screenshots/cors-preflight.png)

Another problem arose after I was able to put and get data to/from the API, but the view was not updating. I added a `console.dir(this.cities)` in the `DataService`, and it was
correctly printing out the populated (local) array of cities, but the view still had an empty array! After looking up this issue, I found the problem: before, I was using LocalStorage to access my
data, which was a `syncronous` process. This meant that when I initiailize my dataservice, the call to `localStorage.getItem()` finished before program execution moved on. When using
the `HttpClient` object, it sends off an HTTP request. This takes some time, and therefore it's *asyncronous*. This involves an Observable, so I had to modify my `getCities()` method
in my `DataService` to correctly handle this. Before I fixed it, it simply returned its local array; but when the Cities component is initialized, for example on initial page load, the HTTP request
for the citites isn't complete, so it returns an empty array. 

My first attempt to solve this problem was this: I have a function `getCityAPI` that returns an `Observable<Array<City>>`. Then, in the component, I subscribe
to that Observable in `ngInit()`, and define a statement to set the component-local cities array to this:
```javascript

ngOnInit() {
this.dataService.getCityAPI().subscribe((data) => {
if(data) this.cities = data;
else this.cities = [];
});
}
```

This appeared to work at first; however, there's a glaring issue: whenever I edit the array through the dataService (i.e. adding a city, restaurant, etc) the view doesn't update anymore. This is
because this retrieval only happens once, and so I needed to reload the page in order to get the updated view. So, I found a second (better) solution: using `get`! Essentially, this binds the
dataService's `cities` array with the *component's* `cities` array. Therefore, when I update the `cities` array in the dataService, that update gets pushed out to interested parties, such as
my components!
```javascript
get cities() {
  return this.dataService.cities;
}
```

It's a very simple and quick way to fix this issue - I just add this to each of my view components, and *voila*! Also, to use `get` I needed to specify target=ts5 in my `tsconfig.json` file.

The next problem was fixing my endpoints: they were saying 'City not found' because the database call hadn't finished yet. To solve this, I needed to adjust the `ngOnInit()` functions.
Instead of checking the data directly from the service, I make an API call to get the cities, then perform the search and decision-making in a `subscribe`. This fixed the endpoints, so it
queried the database first. This is where I added the mat-spinner to indicate the loading.

One problem I had on my backend was that I noticed I was reaching my limit of MongoDB connections. It seemed strange - why did I have over 80 connections.... oh, I forgot to close the
database in my node script, I thought to myself. So, adding `db.close()` whenever I query the database fixed this issue, who knew?

Another problem occured just after I solved the first one: my set/cities endpoint was not accurately setting the data anymore! I realized that I was calling db.close() at the end of the
Express function, but since the `mongoClient.replaceOne()` is asyncronous, the connection was closed before the query could go through! So, I put `db.close()` in the callback
function that is executed once the query goes through.

Lastly, I started to develop my login logic. Everything went well until I wanted to submit the login request when the user hit 'Enter' in the dialog. I tried using `(ngSubmit)` to run the login function,
but the form wouldn't run the function. So, my solution was to add a HostListener directive that listened for the Enter key. If it detected the enter key, it would run the `loginSubmit()` method.
I applied this principle to my other forms, in order to speed up my other forms.

After that, I had no more problems finishing the code and features, and deploying the project to my server. I remembered to add the correct `.htaccess` file in my `exploration3` directory, 
in order to redirect all requests in that directory to the index file (for Angular, this enables routing).

## Sources

https://docs.mongodb.com/manual/reference/method/ - MongoDB's documentation on methods. This helped me find methods such as replaceOne(), insertOne(), etc. to manipulate
my MongoDB.

https://enable-cors.org/server_expressjs.html - I adapted a code template from here in order to enable CORS for my backend API.

https://angular.io/guide/http - I used this official Angular documentation to learn how to use the `HttpClient` module in Angular to make and process AJAX requests.

https://stackoverflow.com/questions/48250528/angular-5-how-do-i-wait-for-my-service-to-finish-and-then-proceed-to-the-next-t?rq=1 - This helped me understand how to use
Observables, both in my `DataService` and components.

https://stackoverflow.com/questions/36208732/angular-2-http-post-is-not-sending-the-request - I learned that I needed to subscribe to Observables from this StackOverflow issue.

https://github.com/TypeStrong/ts-loader/issues/25 - Found out how to use `get` in TypeScript here.

https://stackoverflow.com/questions/43159090/how-can-i-detect-service-variable-change-when-updated-from-another-component - This helped me solve my problem #2: how to 
update component's model when it's changed

https://material.angular.io/components/categories - Official Angular Material Documentation. I used this in this Exploration and in Challenge 4 to research Material components.

https://www.abeautifulsite.net/hashing-passwords-with-nodejs-and-bcrypt - Found this to learn how to use hashing algorithms (such as for passwords) in node.

https://material.io/resources/icons/?style=baseline - used to look up material icons to use in my application UX.

https://esprima.org/demo/validate.html - quick JS syntax checker to help me debug syntax errors in my backend.

https://stackoverflow.com/questions/50010345/angular-stop-enter-key-from-submitting - Used to debug submitting my dialog forms using the enter key, such as logging in.
