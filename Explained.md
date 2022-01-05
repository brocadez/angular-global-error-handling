Global Error Handling in Angular
Learn how to automatically catch all errors in a web application written in Angular and process them accordingly
Philipp Kief
Philipp Kief

Sep 22, 2020·7 min read






A not always so popular but for the end user enormously important topic is the interception of errors. Even if an application has been thoroughly tested before deployment, it is always possible that the user may encounter errors. In this article you will learn how to catch various errors in Angular — one of the world’s most popular front end frameworks — at a global location and process them accordingly.
Error Types
Since nowadays more and more logic is outsourced from the back end to the front end, the probability increases that a faulty behavior of the user leads to an unforeseen error, which can cause the application to “crash”. Now a web application cannot really “crash” like a desktop application, which in the worst case simply closes. No, the web application remains open in the current browser tab, only its behavior is no longer comprehensible to the user in case of an error. Since JavaScript is single-threaded in the browser, it can happen that a part of the interface freezes and an action is not performed correctly. In this case we speak of a local front end error. Since we as the developers don’t know where and when such an error could occur, it is important to catch all occurring errors at a central location.
Another error case that can occur is when a request to the back end fails and the front end receives an error message from the back end. Although in this case it is clear that the error is coming from the back end, there is a need to take care of the error handling for every single request to the back end. Again, it is better to handle these errors in a centralized location so that the user is presented with consistent error messages and also to avoid forgetting to intercept errors.

Example application
In the following we will look at an example application that uses global error handling and go through it step by step. A click on the first two buttons produces an error — the first a local front end error, the second via a bad response from the back end. The third button shows a successful request, where no error message is displayed, but only the loading spinner until the request is completed.

front end error, the second via a bad response from the back end. The third button shows a successful request, where no error message is displayed, but only the loading spinner until the request is completed.

Application architecture
Before we turn our attention to the implementation details, we should also take a look at the structure of our Angular application. In a more complex application it is worthwhile to divide certain functionalities into different modules. For example, as in our case, there is a core module that contains the functionality that is globally available to the whole application and is also loaded immediately when the application is started — such as the error handling.
Another typical module is the shared module. As the name suggests, this module contains functionality that can be reused in several other modules of the application. These are mostly stateless user interface components (or so-called dump components), such as loading spinners or dialogs, which can be controlled by a service. In our case there are two dialogs, one for displaying the error message and one for displaying a loading spinner, which is displayed for the duration of a request.
File structure
In the following a simplified structure of the files is shown. You can directly see the separation into core and shared modules. In this case, the shared directory contains the components and services needed to display the error messages and the loading spinner. The core directory in turn contains the files for the interceptors that globally intercept and process the errors in the application. Both the core module and the shared module are imported into the app module, which is usually the entry module of an application in Angular.

Core Module
The core module is the entry point for the global Error Handler. Two providers are registered here, one for the Error Handler and another for our Loading Spinner. The global error handler catches all errors occurring within our application. The second provider is an HTTP interceptor, which is called for every interaction with the back end. The multi property must be set to true in this case, since the HTTP_INTERCEPTORS injection token can potentially be assigned to several classes.

Global Error Handler
As you have seen in the last section, the GlobalErrorHandler class was registered as provider in the core module. This class implements the ErrorHandler class and contains a handleError method. This method is called whenever an error is thrown somewhere in the application. The error is passed as a parameter and can be processed further inside the method. In our case a dialog is opened where the error message should be displayed and the error is logged to the browser console.
To verify if the error came from an HTTP response we can check if the error object is an instance of the HttpErrorResponse class.

The opening of the dialog takes place in a callback of zone.run, so that the dialog window can be closed even if the error is thrown outside the ngZone. This is for example the case if an error occurs in a lifecycle hook like the ngOnInit function in a component.
Trigger an error
In our example there is a method in the AppComponent called localError which throws an error:
localError() {
  throw Error("The app component has thrown an error!");
}
So, if the first button in the example is clicked it will call this method and the GlobalErrorHandler processes the thrown error by showing this dialog:

The user can then click on the “Close” button to close the error dialog and continue using the application. Of course, the processing of an error may vary from case to case and require further steps. For example, for monitoring purposes, it can be useful to write the error message to a log file in the back end, to navigate the application to another page or to reset a certain state. In this example we simply show an error dialog.

HTTP Loading Interceptor
The interception of HTTP requests is done by an HTTP Interceptor class. The HttpLoadingInterceptor class implements the interface HttpInterceptor by providing the method intercept. This method is called automatically on every HTTP request that is made through our application — regardless of whether it was successful or not. Basically, this allows us to set up a loading spinner. The first thing we do is to call the service for the Loading Spinner dialog to display it:
this.loadingDialogService.openDialog();
The dialog for the loading spinner will be opened and lets the user know that an interaction with the back end is currently taking some time:

After this the processing of the HTTP request is executed which is mainly done with an HTTP handler called next. The HTTP handler provides ahandle method that processes all HTTP requests by returning an RxJS observable. Based on this observable the pipe method can be called which provides us the possibility to use some RxJS operators to handle the requests one after each other.
With the help of the finalize operator the dialog of the loading spinner is hidden again, regardless of whether the request has thrown an error or not.

In our sample application you can see that the following HTTP request procudes a faulty status code (404) and is therefore processed by our global error handler. In addition, a loading spinner is shown until the response from the backend is sent back.
failingRequest() {
  this.http.get("https://httpstat.us/404?sleep=2000").toPromise();
}
The error dialog will look like this:

Conclusion
The approach of this article shows that setting up an overall error handler is of great advantage. Although there is some initial setup effort, in the long run you get a consistent error handling that you don’t have to worry about when adding new features to the application in the future. The same applies to the loading spinner which is automatically displayed for each request. Of course only the basics were shown here, individual adaptations and extensions to this approach are possible, depending on the requirements of the respective application.
Source Code
GitHub - PKief/angular-global-error-handling: Created with StackBlitz ⚡️
Learn how to automatically catch all errors in a web application written in Angular and process them accordingly Run ng…
github.com


