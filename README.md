# Building iOS Native Applications with AWS Amplify

In this workshop we'll learn how to build iOS Mobile Applications with Swift & [AWS Amplify](https://aws-amplify.github.io/).

![](https://imgur.com/IPnnJyf.jpg)

### Topics we'll be covering:

- [Authentication](https://github.com/dennisAWS/aws-amplify-workshop-iOS#adding-authentication)
- [Analytics](https://github.com/dennisAWS/aws-amplify-workshop-iOS#adding-analytics)

## Redeeming our AWS Credit   

1. Visit the [AWS Console](https://console.aws.amazon.com/console).
2. In the top right corner, click on __My Account__.
![](dashboard1.jpg)
3. In the left menu, click __Credits__.
![](dashboard2.jpg)

## Creating a new Xcode Project

Install [Cocoapods](https://cocoapods.org/), if not already installed. From a terminal window navigate into your Xcode project’s application directory and run the following:

```bash
sudo gem install cocoapods
```
Once cocoapods is installed
```bash
pod init
```

### Installing the Amplify CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```

Now we need to configure the CLI with our credentials:

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __eu-central-1__
- Specify the username of the new IAM user: __appdevcon-user__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __appdevcon__

### Initializing A New Amplify Project
From within your Xcode project folder...
```bash
amplify init
```

- Enter a name for the project: __appdevcon__
- Enter a name for the environment: __dev__
- Choose your default editor: __Visual Studio Code (or your default editor)__   
- Please choose the type of app that you're building __iOS__ 
- Do you want to use an AWS profile? __Y__
- Please choose the profile you want to use: __appdevcon__

Now, the AWS Amplify CLI has iniatilized a new project & you will see a new folder: __amplify__. The files in this folder hold your project configuration.

## Adding Authentication (Backend)
For more information on using authentication with iOS. Check out our [iOS authentication docs](https://aws-amplify.github.io/docs/ios/authentication).

To add authentication, we can use the following command:

```sh
amplify add auth
```

> When prompted for __Do you want to use default authentication and security configuration?__, choose __Yes__

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push
```

> To view the new Cognito authentication service at any time after its creation, go to the dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

### Configuring your iOS applicaion for AWS Backend

The first thing we need to do is to configure our iOS Xcode project to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated `awsconfiguration.json` file that is now in the root of our Xcode project folder. 

Launch Xcode. In the Finder, drag `awsconfiguration.json` into Xcode under the top Project Navigator folder (the folder name should match your Xcode project name). When the Options dialog box that appears, do the following:

* Clear the `Copy items if needed` check box.
* Choose `Create groups`, and then choose `Next`.

#### Hot it works
Rather than configuring each service through a constructor or constants file, the AWS SDKs for iOS support configuration through a centralized file called `awsconfiguration.json` which defines all the regions and service endpoints to communicate. Whenever you run `amplify push`, this file is automatically created allowing you to focus on your Swift application code. On iOS projects the `awsconfiguration.json` will be placed into the root directory and you will need to add it to your Xcode project.

## Adding Authentication (iOS Client)
The `AWSMobileClient` provides client APIs and building blocks for developers who want to create user authentication experiences. This includes performing authentication actions, a simple "drop-in auth" UI for performing common tasks, automatic token and credentials management, and state tracking with notifications for performing workflows in your application when users have authenticated.

#### Set up AWS Mobile SDK dependencies (Cocoapods)
After initialization in your project directory with `amplify init`, edit your `Podfile` with the following:
```ruby
target 'MyApp' do             ##Replace MyApp with your application name
  use_frameworks!
  pod 'AWSMobileClient', '~> 2.9.0'      # Required dependency
  pod 'AWSAuthUI', '~> 2.9.0'            # Optional dependency required to use drop-in UI
  pod 'AWSUserPoolsSignIn', '~> 2.9.0'   # Optional dependency required to use drop-in UI
end
```
Pull the SDK libraries into your project:

```terminal
pod install --repo-update
```

Open the **.xcworkspace** file of your project (close the **.xcodeproj** file if you already have it open). Don't forget to drag in the `awsconfiguration.json` file from finder into your **.xcworkspace** if you haven't done so already. Finally, build your project once to ensure all frameworks are pulled in and compile correctly.

#### Initialize the AWSMobileClient in your ViewController
In your root ViewController or related screen that you want users to sign-up or sign-in, add the following code referenced below:

```swift
import AWSMobileClient

    override func viewDidLoad() {
        super.viewDidLoad()
        AWSMobileClient.sharedInstance().initialize { (userState, error) in
            if let userState = userState {
                print("UserState: \(userState.rawValue)")
            } else if let error = error {
                print("error: \(error.localizedDescription)")
            }
        }
    }
```
Build and run your program to see the initialized client in Xcode messages. Since you haven't logged in yet, it will print a state of `signedOut`. The `userState` returns an ENUM which you can perform different actions in your workflow. 
```swift
    @IBOutlet weak var signInStateLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        AWSMobileClient.sharedInstance().initialize { (userState, error) in
            if let userState = userState {
                switch(userState){
                case .signedIn:
                        DispatchQueue.main.async {
                            self.signInStateLabel.text = "Logged In"
                    }
                case .signedOut:
                    AWSMobileClient.sharedInstance().showSignIn(navigationController: self.navigationController!, { (userState, error) in
                            if(error == nil){ //Successful signin
                                DispatchQueue.main.async {
                                    self.signInStateLabel.text = "Logged In"
                                }
                            }
                        })
                default:
                    AWSMobileClient.sharedInstance().signOut()
                }
                
            } else if let error = error {
                print(error.localizedDescription)
            }
        }
    }
```

You might leverage the above workflow to perform other actions in the `signedIn` case, such as calling [GraphQL or REST APIs with AWS AppSync and Amazon API Gateway](./api) or uploading content with [Amazon S3](./storage).

#### Guest access

Many applications have UX with "Guest" or "Unauthenticated" users. This is provided out of the box with `AWSMobileClient` through the initialization routine you have added. However, the Amplify CLI does not enable this by default with the `amplify add auth` flow. You can enable this by running `amplify update auth` and choosing `No, I will setup my own configuration` when prompted. Ensure you choose the **...connected with AWS IAM controls** which will allow you to select **Allow unauthenticated logins**.

When complete run `amplify push` and your `awsconfiguration.json` will work automatically with your updated Cognito settings. The `AWSMobileClient` user session will automatically have permissions configured for Guest/Unauthenticated users upon initialization. 

If you login in your app either using the [Drop-In Auth](#dropinui) or the [direct Auth APIs](#iosapis) then the `AWSMobileClient` user session will transition to an authenticated role.

#### Drop-In Auth

The `AWSMobileClient` client supports easy "drop-in" UI for your application. You can add drop-in Auth UI like so:

```swift
AWSMobileClient.sharedInstance().showSignIn(navigationController: self.navigationController!, { (signInState, error) in
    if let signInState = signInState {
        print("logged in!")
    } else {
        print("error logging in: \(error.localizedDescription)")
    }
})
```
IMPORTANT: The drop-in UI requires the use of a navigation controller to anchor the view controller. Please make sure the app has an active navigation controller which is passed to the `navigationController` parameter.

## Adding Analytics

To add analytics, we can use the following command:

```sh
amplify add analytics
```

> Next, we'll be prompted for the following:

- Provide your pinpoint resource name: __amplifyanalytics__   
- Apps need authorization to send analytics events. Do you want to allow guest/unauthenticated users to send analytics events (recommended when getting started)? __Y__   
- overwrite YOURFILEPATH-cloudformation-template.yml __Y__

### Recording events


