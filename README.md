This tutorial will guide you through writing an iOS app using Realm Swift to sync with the RealmTasks demo apps.

In order to successfully complete this tutorial you will need a Macintosh running macOS 10.12 or later, as well as a copy of Xcode 9.2

The rest of this tutorial will show you how to:

Setup a new Realm-based project from scratch using Cocoapods

How to adopt and setup a free Realm utility module called RealmLoginKit which allows you to easily created multi-user ready applications with almost zero coding

How to create a simple Realm-based Task Manager that can interoperate with the fully-featured RealmTasks that comes with the Realm Mobile Platform distribution

1. Create a new Xcode project
In this section we will create the basic iOS iPhone application skeleton needed for this tutorial.

Launch Xcode 9.

Click "Create a new Xcode project".

Select "iOS", then "Application", then "Single View Application", then click "Next".

Enter "MultiUserRealmTasksTutorial" in the "Product Name" field.

Select "Swift" from the "Language" dropdown menu.

Select "iPhone" from the "Devices" dropdown menu.

Select your team name (log in via Xcode's preferences, if necessary) and enter an organization name.

Click "Next", then select a location on your Mac to create this project, then click "Create".

2. Setting Up Cocoapods
In this section we set up the Cocoapod dependency manager and add Realm's Swift bindings and a utility module that allows us to create a multi-user application using a preconfigured login panel

Quit Xcode

Open a Terminal window and change to the directory/folder where you created the Xcode project

If you have not already done so, install the Cocoapods system 

Full details are available via the Cocopods site, but the simplest instructions are to type sudo gem install cocoapods in a terminal window

Initialize a new Cocoapods Podfile with pod init A new file called Podfile will be created.

Edit the Podfile, find the the comment line that reads:

# Pods for MultiUserRealmTasksTutorial And add the following after this line:

pod 'RealmSwift'
pod 'RealmLoginKit'
Save the file

At the terminal, type pod install - this will cause the Cocoapods system to fetch the RealmSwift and RealmLoginKit modules, as well as create a new Xcode workspace file which enabled these modules to be used in this project.

3. Setting up the Application Delegate
Reopen Xcode, but rather than open xcodeproj use the newly created xcworkspace file; this was created by the Cocoapods dependency manager and should be used going forward

If you have not already, open the xcworkspace with Xcode.

In this section we will configure the application delegate to support a Navigation controller. From the Project Navigator, double-clock the AppDelegate.swift file and edit the file to replace the AppDelegate class with the following:

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions:[UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = UINavigationController(rootViewController: TasksLoginViewController())
        window?.makeKeyAndVisible()
        return true
    }
}
4. Setting Up the Storyboard & Views
In this section we will set up our login and main view controller's storyboard connections.

In the Xcode project navigator select the main.storyboard file. Interface builder (IB) will open and show the default single view layout:



 



2. Adding the TableViewController - on the lower right of the window is the object browser, type "tableview" to narrow down the possible IB objects. There will be a "TableView Controller" object visible. Drag this onto the canvas. Once you have done this the Storyboard view will resemble this:



 



Once you have added the second view controller, you will need to add class names/storyboard IDs for each controller to prepare for the code will be adding in the next sections:

Open the storyboard propery viewer to see the ourline view of the contents of both controllers in the sotoryboard. Then, control-drag from the TasksLoginViewController label to the Table View Controller label and select "show" when the popup menu appears.

You will need to set the class names for each of the view controller objects. To do this select the controllers one at a time, and for the LoginView Controller, set the class name to TasksLoginViewController and to the storyboard id to loginView. For the new TableViewController you added, set the class name to TasksTableViewController and here set the storyboard id to tasksView.

5. Create the Models and Constants Class File
In this step we are going to create a few constants to help us manage our Realm as well as the class models our Realm will operate on.

From the Project Navigator, right click and select New File and when the file selector apprears select Swift Fileand name the file ConstantsAndModels and press return. Xcode will create a new Swift file and open it in the editor.

Let's start with the Constants; add the following to the file:

import Foundation
import RealmSwift

struct Constants {

    // the host that will do the synch - if oure using the Mac dev kit you probably want this to be localhost/127.0.0.1
    // if you are using the Professional or Enterprise Editions, then this will be a host on the Internet
    static let defaultSyncHost                      = "127.0.0.1"
    
    // this is purely for talking to the RMP auth system
    static let syncAuthURL                          = URL(string: "http://\(defaultSyncHost):9080")!
    
    // The following URLs and URI fragments are about talking to the synchronization service and the Realms
    // it manages on behalf of your application:
    static let syncServerURL                        = URL(string: "realm://\(defaultSyncHost):9080/")
    
    // Note: When we say Realm file we mean literally the entire collection of models/schemas inside that Realm...
    // So we need to be very clear what models that are represented by a given Realm.  For example:
    
    // this is a realm where we can store profile info - not covered in the main line of this tutorial
    static let commonRealmURL                       = URL(string: "realm://\(defaultSyncHost):9080/CommonRealm")!
    
    // Note: If Swift supported C-style macros, we could simply define the configuration for the tasks Realm like this:
    //
    
    //  this is a task Realm comptible with the fully version of RealmTasks for iOS/Android/C#
    static let tasksRealmURL                       = URL(string: "realm://\(defaultSyncHost):9080/~/realmtasks")!
    
    //  static let tasksRealmConfig                    = Realm.Configuration(syncConfiguration: SyncConfiguration(user: SyncUser.current!, realmURL:    tasksRealmURL),objectTypes: [TaskList.self, Task.self])
}

func tasksRealmConfig(user: SyncUser) -> Realm.Configuration  {
    let config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: SyncUser.current!, realmURL: Constants.tasksRealmURL), objectTypes: [TaskList.self, Task.self])
    return config
}


There key things to note here are how the contents are layered together to create a set of accessors that allow you to quickly and easily create references to a Realm. This example shows a single Realm but in more complex projects one could imagine having a number of such accessors created for a number of special purpose Realms.

Next, we'll add the definitions of our models. Note that there are two kinds of models here: the Task and taskList models.


// MARK: Model

final class TaskList: Object {
    @objc dynamic var text = ""
    @objc dynamic var id = ""
    let items = List<Task>()
    
    override static func primaryKey() -> String? {
        return "id"
    }
}

final class Task: Object {
    @objc dynamic var text = ""
    @objc dynamic var completed = false
    @objc dynamic var duedate: Date?
}

At this point, we've created a login system, and defined the data models (Task and TaskList) that we'll use to represent our data and sync with the RealmTasks apps.

6. Configuring the Login View Controller
In this section we will rename and then configure the TasksLoginViewController that will allow you to log in an existing user account, or create a new account

Open the ViewController.swift file in the project navigator. Click once on it to enable editing of the file name; change the name to TasksLoginViewController and press return to rename the file.

Clicking on the filename should also have opened the newly renamed file in the editor. Here too you should replace all references to ViewController in the comments and class name with TasksLoginViewController

Next, we are going to update the contents of this view controller and take it from a generic, empty controller to one that can display our Login Panel.

Start by modifying the imports to read as follows:

import UIKit
import RealmSwift
import RealmLoginKit
Modify the top of the class file so the following properties are declared:


class TasksLoginViewController: UITableViewController {
var loginViewController: LoginViewController!
var token: NotificationToken!
var myIdentity = SyncUser.current?.identity!

Next, create the viewWillAppear method to

       override func viewDidAppear(_ animated: Bool) {
       loginViewController = LoginViewController(style: .lightOpaque)
       loginViewController.isServerURLFieldHidden = false
       loginViewController.isRegistering = true
       
       if (SyncUser.current != nil) {
           // yup - we've got a stored session, so just go right to the UITabView
           Realm.Configuration.defaultConfiguration = tasksRealmConfig(user: SyncUser.current!)
           
           self.navigationController?.setViewControllers([TasksTableViewController()], animated: true)
           self.loginViewController!.dismiss(animated: true, completion: nil)
       } else {
           // show the RealmLoginKit controller
           if loginViewController!.serverURL == nil {
               loginViewController!.serverURL = Constants.syncAuthURL.absoluteString
           }
           // Set a closure that will be called on successful login
           loginViewController.loginSuccessfulHandler = { user in
               DispatchQueue.main.async {
                   // this AsyncOpen call will open the described Realm and wait for it to download before calling its closure
                   Realm.asyncOpen(configuration: tasksRealmConfig(user: SyncUser.current!)) { realm, error in
                       if realm != nil {
                           Realm.Configuration.defaultConfiguration = tasksRealmConfig(user: SyncUser.current!)

                           //self.loginViewController!.dismiss(animated: true, completion: nil)
                           
                           // let's instantiate the nexty view controller...
                           let tasklistVC = TasksTableViewController()
                           tasklistVC.realm = realm //  and set up its realm reference (since have it).
                           
                           // then we can set the nav controller to use this view and then dismiss the login view
                           self.navigationController?.setViewControllers([tasklistVC], animated: true)
                           self.loginViewController!.dismiss(animated: true, completion: nil)
                           
                       } else if let error = error {
                           print("An error occurred while logging in: \(error.localizedDescription)")
                       }
                   } // of asyncOpen()
                   
               } // of main queue dispatch
           }// of login controller
           
           present(loginViewController, animated: true, completion: nil)
       }
} 
7. The TaskList Controller: Add a title and register a cell class for use with our table view
In this section we will create and configure our TasksTableViewController

Create a new file agin. This time you will select a "Cocoa Touch" class, then press "Next" For the class section you want to enter TasksTableViewController which is the name you entered when you created and configured the view controller in the storyboard; for Subclass of you want to enter `UITableViewController` (typing the first few characters will cause Xcode to help with autocompletion suggestions).



 



The language should be set to "Swift. Click "next" and save the file alond side the other files in this project.

Xcode will open the file and we can now start by adding Realm to our import statement

import RealmSwift
Add the following property to your TasksTableViewController class, on a new line, right after the class declaration:

    var realm:          Realm!
    var items:          List<Task>?         // all of the tasks
    var myTaskLists:    Results<TaskList>?  // all tasks lists
    var currentTaskList: TaskList?          // the current list we're watching
    var notificationToken: NotificationToken!
Next, we'll have a method that configures the Table View when the controller loads. Edit the ViewDidLoad method as follows:

override func viewDidLoad() {
    super.viewDidLoad()
    setupUI()
}


func setupUI() {  
    self.title = "My Tasks"
    tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
}
Append the following to the end of your TasksTableViewController class's body:

// MARK: Turn off the staus bar

open override var prefersStatusBarHidden : Bool {
    return true
}


// MARK: UITableView

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
         return self.items!.count
    }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        let item = self.items![indexPath.row]
        cell.textLabel?.text = item.text
        cell.textLabel?.alpha = item.completed ? 0.5 : 1
        print("working on cell for row: \(indexPath.row), text is \"\(item.text)\"")
        return cell
    }
If you then build and run the app, you'll see your one task being displayed in the table. There's also some code in here to show completed items as a little lighter than uncompleted items, but we won't see that in action until later.

Now lets add support for Realm:

func setupRealm() {
        // Open the Realm, if necessary - it's probably been passed in to us
        // by the login mechanism which did an realm.asyncOpen()
        if self.realm == nil {
            self.realm = try! Realm(configuration: tasksRealmConfig(user: SyncUser.current!))
        }
        
        // Next, let's get all of our task lists, and then we'll set the default to the first one
        // (of course if there are not, in fact, any taks lists, we'll make one)
        self.myTaskLists = self.realm.objects(TaskList.self)
        if self.myTaskLists!.count > 0 {
            self.currentTaskList = self.myTaskLists!.first
        } else {
            try! realm.write {
                let initialValues = ["id": NSUUID().uuidString, "text": "My First Task List"]
                let newTaskList = realm.create(TaskList.self, value: initialValues)
                self.currentTaskList = newTaskList
                
                let values =  ["text": "My First Task"]
                let newTask = realm.create(Task.self, value: values)
                realm.add(newTask) // add the new tasks
                self.currentTaskList?.items.append(newTask) // and add it to our default list
            }
        }
        
        // Now, get all of our tasks, if any.  On return, we'll check to see if the list is empty
        // and make a prototype 1st task for the user so they're not looking ast a blank screen.
        self.items = self.currentTaskList?.items // NB: this will return an empty list if there are no tasks
        
        // and, finally, let's listen for changes to the task items
        self.notificationToken = self.setupNotifications()

    }// of setupRealm
You may have noticed that this code will not run without the setupNotifications function, let's add that here:

    func setupNotifications() -> NotificationToken? {
        return self.currentTaskList?.items.observe { [weak self] (changes: RealmCollectionChange) in
            guard let tableView = self?.tableView else { return }
            switch changes {
            case .initial:
                // Results are now populated and can be accessed without blocking the UI
                tableView.reloadData()
            case .update(_, let deletions, let insertions, let modifications):
                // Query results have changed, so apply them to the UITableView
                tableView.beginUpdates()
                tableView.insertRows(at: insertions.map({ IndexPath(row: $0, section: 0) }),
                                     with: .automatic)
                tableView.deleteRows(at: deletions.map({ IndexPath(row: $0, section: 0)}),
                                     with: .automatic)
                tableView.reloadRows(at: modifications.map({ IndexPath(row: $0, section: 0) }),
                                     with: .automatic)
                tableView.endUpdates()
            case .error(let error):
                // An error occurred while opening the Realm file on the background worker thread
                fatalError("\(error)")
            }
        }
    } // of setupNotifications
It is also important to de-initialize once the class is deallocated:    

    deinit {
        notificationToken?.invalidate()
    }
8. Add support for creating new tasks
Add the following function to your TasksTableViewController class at its end:

// MARK: Functions

    @objc func add() {
        let alertController = UIAlertController(title: "New Task", message: "Enter Task Name", preferredStyle: .alert)
        var alertTextField: UITextField!
        alertController.addTextField { textField in
            alertTextField = textField
            textField.placeholder = "Task Name"
        }
        alertController.addAction(UIAlertAction(title: "Add", style: .default) { _ in
            guard let text = alertTextField.text , !text.isEmpty else { return }
            
            let items = self.items
            try! items?.realm?.write {
                items!.insert(Task(value: ["text": text]), at: items!.filter("completed = false").count)
            }
        })
        present(alertController, animated: true, completion: nil)
    }
Now add the following to the setupUI() function so it looks as follows:

func setupUI() {
        title = "My Tasks"
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        
        let addButton = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(add))
        let logoutButton = UIBarButtonItem(title: NSLocalizedString("Logout", comment:"logout"), style: UIBarButtonItemStyle.plain, target: self, action: #selector(handleLogout))
        
        self.navigationItem.rightBarButtonItems = [addButton, logoutButton]
        self.navigationItem.leftBarButtonItem = editButtonItem

    }
Now lets add the logout functionality: 

    // Logout Support
    
    
    @IBAction  func handleLogout(sender:AnyObject?) {
        let alert = UIAlertController(title: NSLocalizedString("Logout", comment: "Logout"), message: NSLocalizedString("Really Log Out?", comment: "Really Log Out?"), preferredStyle: .alert)
        
        // Logout button
        let OKAction = UIAlertAction(title: NSLocalizedString("Logout", comment: "logout"), style: .default) { (action:UIAlertAction!) in
            SyncUser.current?.logOut()
            self.navigationController?.setViewControllers([TasksLoginViewController()], animated: true)
        }
        alert.addAction(OKAction)
        
        // Cancel button
        let cancelAction = UIAlertAction(title: "Cancel", style: .cancel) { (action:UIAlertAction!) in
            print("Cancel button tapped");
        }
        alert.addAction(cancelAction)
        
        // Present Dialog message
        present(alert, animated: true, completion:nil)
    }
Lastly, we need to allow non-TLS network requests to talk to our sync server.

Right-click on the Info.plist file and select "Open as... Source Code" and paste the following in the <dict>section:

<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
If you build and run the app now, it should connect to the object server and display the tasks that were added in RealmTasks earlier.

If you add new tasks by tapping the "Add" button in your app, you should immediately see them reflected in the RealmTasks app too.



 



Congratulations, you've built your first synced Realm app!

Keep going if you'd like to see how easy it is to add more functionality and finish building your task management app.

10. Support moving and deleting tasks
Add the following right after these tableView overrides in your UITableView section:

    override func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
        try! self.items?.realm?.write {
            self.items?.move(from: sourceIndexPath.row, to: destinationIndexPath.row)
        }
    }
    
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            try! realm.write {
                let item = self.items?[indexPath.row]
                realm.delete(item!)
            }
        }
    }

11. Support toggling the 'completed' state of a task by tapping it
After the last tableView function in the TasksTableViewController class, add the following function override:

    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let item = items![indexPath.row]
        try! item.realm?.write {
            item.completed = !item.completed
            let destinationIndexPath: IndexPath
            if item.completed {
                // move cell to bottom
                destinationIndexPath = IndexPath(row: items!.count - 1, section: 0)
            } else {
                // move cell just above the first completed item
                let completedCount = items!.filter("completed = true").count
                destinationIndexPath = IndexPath(row: items!.count - completedCount - 1, section: 0)
            }
            self.items!.move(from: indexPath.row, to: destinationIndexPath.row)
        }
    }


12. Spin up a Remote Realm Object Server and Connect your App
Now let's launch a cloud instance on AWS EC2 and install Realm Object Server

 

SSH to the EC2 install and use the following script to install ROS with all its dependencies

curl -s https://raw.githubusercontent.com/realm/realm-object-server/master/install.sh | bash
Once complete you then run:

 ros init myROS
Which will generate a template ROS folder - you can now step into that folder and run:

npm install
npm start
Now edit the ConstantsAndModels.swift file in your XCode to connect to the new server

13. Connect with Notifier Code
Now back on your laptop create a new Javascript file in your favorite IDE in a folder of your choice with the below code:

'use strict';

var fs = require('fs');
var Realm = require('realm');
const {Wit, log} = require('node-wit');

//TODO Generate an expiring token and add all todo 

// Professional or Enterprise Edition access token
var token = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJSZWFsbSIsIk5vdGlmaWVyIjp0cnVlLCJTeW5jIjp0cnVlLCJpYXQiOjE1MTM5MDM0NDYsImV4cCI6MTUxNDUwODI0Nn0.G7yFCMrS66R44scHYRoLgRQYFrLYwjFo2NXgVzOEzkZ7xII37Va4G8XmhFM9fGazgpixIAYVvOO4p6AUmxwerS4K7YnnjNsNAQh1-C_rSQc7Dw6QAtN-rJO5z0BwfDkQOvKmxH69Cl397sg3GmmfLTSz6hG-LhwCzY0GWBjkHhmE30JN79CxwpdYvxaUQ2BlomT7BYnZ4tvxKmlaXlKoUKIrCIBhBvHG55VdtXuEDmX8lfE7ku2raqMYC1vO3bevZFU6D0FDTPb97KJXa9Rn-IEhqAj4Rh91NAxf2KaLCiPPS2LVc_YDgCb6XqYolNtczZ_eOAwfLxNTa9DglZW8kw";
Realm.Sync.setFeatureToken(token);

// Realm admin token
//THIS INFORMATION WILL BE PROVIDED BY INSTRUCTOR
var REALM_ADMIN_TOKEN = "INSERT_REALM_ADMIN_TOKEN";

// Server access token from wit.ai API
var WIT_ACCESS_TOKEN = "TXO5QPZWCCDN7WBWECGGJR7NS4PM7V63";

// The URL to the Realm Object Server
//format should be: 'realm://IP_ADDRESS:port'  like example below 
// var SERVER_URL = 'realm://127.0.0.1:9080';
var SERVER_URL = 'INSERT_SERVER_URL';

//format should be: 'http://IP_ADDRESS:port')
//var ADMIN_URL = 'http://127.0.0.1:9080';
var ADMIN_URL = 'INSERT_ADMIN_URL';

// The path used by the global notifier to listen for changes across all
// Realms that match.
// Find your user ID by using Realm Studio to browse for user info 
var NOTIFIER_PATH = "/INSERT_USER_ID/realmtasks";

const client = new Wit({accessToken: WIT_ACCESS_TOKEN})

//enable debugging if needed 
//Realm.Sync.setLogLevel('debug');

const adminUser = Realm.Sync.User.adminUser(REALM_ADMIN_TOKEN, ADMIN_URL);

//INSERT HANDLER HERE
var change_notification_callback = function(changeEvent) {
    const realm = changeEvent.realm;
    const tasks = realm.objects('Task');

    if (!changeEvent.changes.Task) {
        return // don't do anything nothing changed
    }

    console.log('received change event');        

    const taskInsertIndexes = changeEvent.changes.Task.insertions;
    const taskModIndexes = changeEvent.changes.Task.modifications;
    const taskDeleteIndexes = changeEvent.changes.Task.deletions;

    for (var i = 0; i < taskInsertIndexes.length; i++) {
        const task = tasks[taskInsertIndexes[i]];
        if (task !== undefined) {
            console.log('insertion occurred');
            const client = new Wit({accessToken: WIT_ACCESS_TOKEN});
            
            client.message(task.text, {}).then((data) => {
                console.log("Response received from wit: " + JSON.stringify(data));
                if(data.entities.datetime){
                    var dateTime = data.entities.datetime[0];
                    if (!dateTime) {
                        console.log("Couldn't find a date.");
                        return;
                    }
                    console.log("Isolated calculated date: " + dateTime.value);
                    
                    realm.write(function() {
                        console.log(dateTime.value);
                        task.duedate = dateTime.value;
                });
                }
                
            }).catch(console.log);    
        }
    }

    for (var i = 0; i < taskModIndexes.length; i++) {
        const task = tasks[taskModIndexes[i]];
        if (task !== undefined) {
            console.log('modification occurred');
        }
    }
    
    //when task is deleted
    for (var i = 0; i < taskDeleteIndexes.length; i++) {
        const task = tasks[taskDeleteIndexes[i]];
        if (task !== undefined) {
            console.log('deletion occurred');
        }
    }

}


Realm.Sync.addListener(SERVER_URL, adminUser, NOTIFIER_PATH, 'change', change_notification_callback);

console.log('listening');

Don't forget to run:

npm install realm
npm install node-wit
Then your script with the appropriate fields filled in. You can find the userId by taking a look on Studio.



Congratulations!
