# RealmTraining
Training for the Realm Object Server

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

Open a Terminal window and change to the directory/folder where you created the Xcode RealmTasksTutorialproject

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
In this seciton we will configure the applicaiton degelgate to support a Navigation controller. From the Project Navigator, double-clock the AppDelegate.swift file and edit the file to replace the func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions method with the following:

class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions:[UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = UINavigationController(rootViewController: UIViewController())
        window?.makeKeyAndVisible()
        return true
    }
}
4. Setting Up the Storyboard & Views
In this section we will set up our login and main view controller's storyboard connections.

Reopen Xcode, but rather than open MultiUserRealmTasksTutorial.xcodeproj use the newly created MultiUserRealmTasksTutorial.xcworkspace file; this was created by the Cocoapods dependency manager and should be used going forward

If you have not already, open the MultiUserRealmTasksTutorial.xcworkspace with Xcode.

In the Xcode project navigator select the main.storyboard file. Interface builder (IB) will open and show the default single view layout:






Adding the TableViewController - on the lower right of the window is the object browser, type "tableview" to narrow down the possible IB objects. There will be a "TableView Controller" object visible. Drag this onto the canvas. Once you have done this the Storyboard view will resemble this:






Once you have added the second view controller, you will need to connect the two controllers by a pair of segues, as well as add class names/storyboard IDs for each controller to prepare for the code will be adding in the next sections:

Open the storyboard property viewer to see the outline view of the contents of both controllers in the storyboard.

You will need to set the class names for each of the view controller objects. To do this select the controllers one at a time, and for the LoginView Controller, set the class name to TasksLoginViewController and to the storyboard id to loginView. For the new TableViewController you added, set the class name to TasksTableViewController and here set the storyboard id to tasksView

Then, control-drag from the TasksLoginViewController label to the TasksTableViewController label and select "show" when the popup menu appears. Select the segue that is created between the two controllers, and set the name of the segue in the property view on the right side to loginToTasksViewSegue

Do the same from the TasksLoginViewController back to the TasksLoginViewController. Here, again, tap the newly created segue (it will be the diagonal line) and name this segue tasksViewToLoginControllerSegue



The final configfuration will look like this:






5. Configuring the Login View Controller
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
Next, modify the empty viewWillAppear method to look like this:

 override func viewDidAppear(_ animated: Bool) {
     loginViewController = LoginViewController(style: .lightOpaque)
     loginViewController.isServerURLFieldHidden = false
     loginViewController.isRegistering = true

     if (SyncUser.current != nil) {
         // yup - we've got a stored session, so just go right to the UITabView
         Realm.Configuration.defaultConfiguration = commonRealmConfig(user: SyncUser.current!)

         performSegue(withIdentifier: Constants.kLoginToMainView, sender: self)
     } else {
         // show the RealmLoginKit controller
         if loginViewController!.serverURL == nil {
             loginViewController!.serverURL = Constants.syncAuthURL.absoluteString
         }
         // Set a closure that will be called on successful login
         loginViewController.loginSuccessfulHandler = { user in
             DispatchQueue.main.async {
                 // this AsyncOpen call will open the described Realm and wait for it to download before calling its closure
                 Realm.asyncOpen(configuration: commonRealmConfig(user: SyncUser.current!)) { realm, error in
                     if let realm = realm {
                         Realm.Configuration.defaultConfiguration = commonRealmConfig(user: SyncUser.current!)
                         self.loginViewController!.dismiss(animated: true, completion: nil)
                         self.performSegue(withIdentifier: Constants.kLoginToMainView, sender: nil)

                     } else if let error = error {
                         print("An error occurred while logging in: \(error.localizedDescription)")
                     }
                 } // of asyncOpen()

             } // of main queue dispatch
         }// of login controller

         present(loginViewController, animated: true, completion: nil)
     }
 }
Optionally, commit your progress in source control.

Your app should now build and run---although so far it doesn't do much, it will show you to login panel you just configured:






Click the stop button to terminate the app, and we will continue with the rest of the changes needed to create our Realm Tasks app.

6. Create the Models and Constants Class File
In this step we are going to create a few constants to help us manage our Realm as well as the class models our Realm will operate on.

From the Project Navigator, right click and select New File and when the file selector appears select Swift Fileand name the file ConstantsAndModels and press return. Xcode will create a new Swift file and open it in the editor.

Our first task will be to create some constants that will make opening and working with Realms easier, then we will define the Task models. To do this add the following code to the file

Let's start with the Constants, add the following to the file:

import Foundation
import RealmSwift
struct Constants {
    // segue names
    static let      kLoginToMainView                = "loginToTasksViewSegue"
    static let      kExitToLoginViewSegue           = "tasksToLoginViewSegue"


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
    //static let commonRealmConfig                    = Realm.Configuration(syncConfiguration: SyncConfiguration(user: SyncUser.current!, realmURL: commonRealmURL),objectTypes: [Person.self])
    // However the key bit of information the Realm config needs is which user (e.g., SyncUser) it's being configured with.  As a static
    // this would get set only once at app launch time -- so if your initial user logs out and someone else tries to log in, the
    // configuration would still be using the SyncUser.currrent value obtained at launch. So, instead we'll use the function below which
    // obtains the SyncUser dyanamically.  This same logic appplies to other Realm configs like tasksRealmConfig too.

    //  this is a task Realm comptible with the fully version of RealmTasks for iOS/Android/C#
    static let tasksRealmURL                       = URL(string: "realm://\(defaultSyncHost):9080/~/realmtasks")!

    //  static let tasksRealmConfig                    = Realm.Configuration(syncConfiguration: SyncConfiguration(user: SyncUser.current!, realmURL:    tasksRealmURL),objectTypes: [TaskList.self, Task.self])
}

func commonRealmConfig(user: SyncUser) -> Realm.Configuration  {
    let config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: SyncUser.current!, realmURL: Constants.commonRealmURL), objectTypes: [Person.self])
    return config
}

func tasksRealmConfig(user: SyncUser) -> Realm.Configuration  {
    let config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: SyncUser.current!, realmURL: Constants.commonRealmURL), objectTypes: [TaskList.self, Task.self])
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
}

class Person : Object {
    @objc dynamic var id = ""
    @objc dynamic var creationDate: Date?
    @objc dynamic var lastUpdated: Date?
    
    @objc dynamic var lastName = ""
    @objc dynamic var firstName = ""
    
    
    override static func primaryKey() -> String? {
        return "id"
    }
    
    func fullName() -> String {
        return "\(firstName) \(lastName)"
    }
    
    
    
    class func createProfile() -> Person? {
        let commomRealm =  try! Realm(configuration: commonRealmConfig(user: SyncUser.current!))
        var profileRecord = commomRealm.objects(Person.self).filter(NSPredicate(format: "id = %@", SyncUser.current!.identity!)).first
        if profileRecord == nil {
            try! commomRealm.write {
                profileRecord = commomRealm.create(Person.self, value:["id": SyncUser.current!.identity!, "creationDate": Date(),  "lastUpdated": Date()])
                commomRealm.add(profileRecord!, update: true)
            }
        }
        return profileRecord
    }
} // of Person
At this point, we've created a login system, and defined the data models (Task and TaskList) that we'll use to represent our data and sync with the RealmTasks apps.

Your app should still build and run.

7. The TaskList Controller: Add a title and register a cell class for use with our table view
In this section we will create and configure our `TasksTableViewController`

In the project navigator, right-click on the MultiUserRealmTasksTutorial group and select new file agin. This time you will select a "Cocoa Touch" class, then press "Next" For the class section you want to enter TasksTableViewController which is the name you entered when you created and configured the view controller in the storyboard; for Subclass of you want to enter `UITableViewController` (typing the first few characters will cause Xcode to help with autocompletion suggestions).






The language should be set to "Swift. Click "next" and save the file alond side the other files in this project.

Xcode will open the file and we can now start configuring this view controller by replacing or editing the methods as indicated below:

Use a Realm List to references Tasks in the table view:

Add the following property to your ViewController class, on a new line, right after the class declaration:

You will also need:

import RealmSwift
var items = List<Task>()
Next, we'll have a method that configures the Table View when the controller loads. Edit the ViewDidLoad method as follows:

override func viewDidLoad() {
    super.viewDidLoad()
    setupUI()
}

    func setupUI() {
        self.title = "My Tasks"
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
    }

Add the following line to the end of your viewDidLoad() function to seed the list with some initial data:

override func viewDidLoad() {
    // ... existing function ...
    items.append(Task(value: ["text": "My First Task"]))
}
Append the following to the end of your ViewController class's body:

// MARK: Turn off the staus bar

open override var prefersStatusBarHidden : Bool {
    return true
}


// MARK: UITableView

override func tableView(_ tableView: UITableView?, numberOfRowsInSection section: Int) -> Int {
    return items.count
}

override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
    let item = items[indexPath.row]
    cell.textLabel?.text = item.text
    cell.textLabel?.alpha = item.completed ? 0.5 : 1
    return cell
}
If you then build and run the app, you'll see your one task being displayed in the table. There's also some code in here to show completed items as a little lighter than uncompleted items, but we won't see that in action until later.

8. Add support for creating new tasks
Delete the line in your viewDidLoad() function that seeded initial data

DELETE

override func viewDidLoad() {
    // -- DELETE THE FOLLOWING LINE --
    items.append(Task(value: ["text": "My First Task"]))
}
Add the following function to your ViewController class at its end:

// MARK: Functions

func add() {
    let alertController = UIAlertController(title: "New Task", message: "Enter Task Name", preferredStyle: .alert)
    var alertTextField: UITextField!
    alertController.addTextField { textField in
        alertTextField = textField
        textField.placeholder = "Task Name"
    }
    alertController.addAction(UIAlertAction(title: "Add", style: .default) { _ in
        guard let text = alertTextField.text , !text.isEmpty else { return }

        self.items.append(Task(value: ["text": text]))
        self.tableView.reloadData()
    })
    present(alertController, animated: true, completion: nil)
}
Now add the following line at the end of the setupUI() function:

func setupUI() {
    // ... existing function ...

    // we don't have a UINavigationController so let's add a hand-constructed UINavBar
    let screenSize: CGRect = UIScreen.main.bounds
    let navBar = UINavigationBar(frame: CGRect(x: 0, y: 0, width: screenSize.width, height: 44))
    let navItem = UINavigationItem(title: "")
    let addButton = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(add))
    let logoutButton = UIBarButtonItem(title: NSLocalizedString("Logout", comment:"logout"), style: UIBarButtonItemStyle.plain, target: self, action: #selector(handleLogout))
    navItem.leftBarButtonItem = editButtonItem
    navBar.setItems([navItem], animated: false)

    self.view.addSubview(navBar)
}
9. Back items by a Realm and integrate sync
Now, add the following properties to your ViewController class, just under the items property:

var notificationToken: NotificationToken!
var realm: Realm!
The notification token we just added will be needed when we start observing changes from the Realm.

Right after the end of the setupUI() function, add the following:

deinit {
    notificationToken.stop()
}
/// DELETE??? Needed??? In the code above, enter the same values for the username and password variables as you used when registering a user through the RealmTasks app in the "Getting Started" steps.

Then insert the following at the end of the setupRealm() function (inside the function body):

func setupRealm() {
    DispatchQueue.main.async {
        // Open Realm
        self.realm = try! Realm(configuration: tasksRealmConfig(user: SyncUser.current!))

        // Show initial tasks
        func updateList() {
            if self.items.realm == nil, let list = self.realm.objects(TaskList.self).first {
                self.items = list.items
            }
            self.tableView.reloadData()
        }
        updateList()

        // Notify us when Realm changes
        self.notificationToken = self.realm.addNotificationBlock { _ in
            updateList()
        }
    } // of Dispatch...main
}// of setupRealm
And, call this setup function at the end of the viewDidLoad() function:

override func viewDidLoad() {
    // ... existing function ...
    setupRealm()
}




DELETE PREVIOUS??

Now edit the add() function to look like this:

func add() {
    let alertController = UIAlertController(title: "New Task", message: "Enter Task Name", preferredStyle: .alert)
    var alertTextField: UITextField!
    alertController.addTextField { textField in
        alertTextField = textField
        textField.placeholder = "Task Name"
    }
    alertController.addAction(UIAlertAction(title: "Add", style: .default) { _ in
        guard let text = alertTextField.text , !text.isEmpty else { return }

        let items = self.items
        try! items.realm?.write {
            items.insert(Task(value: ["text": text]), at: items.filter("completed = false").count)
        }
    })
    present(alertController, animated: true, completion: nil)
}
This deletes two lines in the guard block that begin with self. and replaces them with a let and try! block which will actually write a new task to the Realm.

Lastly, we need to allow non-TLS network requests to talk to our local sync server.

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
Add the following right after the navItem.rightBarButtonItems like in your setupUI() function:

func setupUI() {
    // ... existing function ...
        navItem.leftBarButtonItem = editButtonItem
    // ... existing function ...
}
This adds the Edit button to the navigation bar.

Now, add these functions to the ViewController class body, right after the other tableView functions:

override func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {
    try! items.realm?.write {
        items.move(from: sourceIndexPath.row, to: destinationIndexPath.row)
    }
}

override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
    if editingStyle == .delete {
          try! realm.write {
              let item = items[indexPath.row]
              realm.delete(item)
          }
    }
}
11. Support toggling the 'completed' state of a task by tapping it
After the last tableView function in the ViewController class, add the following function override:

override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let item = items[indexPath.row]
    try! item.realm?.write {
        item.completed = !item.completed
        let destinationIndexPath: IndexPath
        if item.completed {
            // move cell to bottom
            destinationIndexPath = IndexPath(row: items.count - 1, section: 0)
        } else {
            // move cell just above the first completed item
            let completedCount = items.filter("completed = true").count
            destinationIndexPath = IndexPath(row: items.count - completedCount - 1, section: 0)
        }
        items.move(from: indexPath.row, to: destinationIndexPath.row)
    }
}
12. Adding a Logout Capabilities
In this section we're going to add a logoout capability Add the following two methods to the bottom of the TaskViewController class

// Logout Support


@IBAction  func handleLogout(sender:AnyObject?) {
    let alert = UIAlertController(title: NSLocalizedString("Logout", comment: "Logout"), message: NSLocalizedString("Really Log Out?", comment: "Really Log Out?"), preferredStyle: .alert)

    // Logout button
    let OKAction = UIAlertAction(title: NSLocalizedString("Logout", comment: "logout"), style: .default) { (action:UIAlertAction!) in
        SyncUser.current?.logOut()
        //Now we need to segue to the login view controller
        self.performSegue(withIdentifier: Constants.kExitToLoginViewSegue, sender: self)
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
this code supports the logot button in the TaskTable View navigation controller and will log theuser out and take them back to the RealmLoginKit panel.


13. You're done!```
