## Version Control for Media-Rich Systems
Note: this tutorial assumes that you have some experience using version control software. If that’s not the case, please contact me so we can cover some basics together. 

While versioning text is a (somewhat) solved problem, versioning binary files is much more difficult. There is no way to effectively diff or merge binary files, which means we’re left with completely replacing binary files from one version to the next. A few different source control management (SCM aka Version Control Systems or VCS) systems can help us with the process, and manage binary files between teams of users. 

The two we’ll discuss in this class are Git (with the git-lfs extension) and Perforce. There are good reasons for choosing both, the list below is by no means to be considered comprehensive. If this influences your decision at all, I will be more useful as a resource for Git than Perforce… long live open-source software!

### Pros for Git
• **It’s open source and free to use**. Anyone can setup their own git server, although companies like GitHub, BitBucket, and Assembla have made this largely unnecessary.  
• Widely used across multiple industries and academic fields.  
• Distributed repositories by default.  

### Pros for Perforce (aka HelixCore, because, marketing I guess)
• Widely used at AAA studios. Because Perforce got a jump on handling binary files in a reasonable way, it quickly gained mindshare among game studios.   
• Centralized repository by default. Helps to keep everything in sync. You can also create a personal server for local commits.  
• Game Editor (Unreal, Unity etc.) support is typically a bit more robust than it is for Git.  

Here’s [a very biased (and there’s some actual misinformation in there) comparison from Perforce](https://www.perforce.com/blog/vcs/git-vs-perforce-how-choose-and-when-use-both)

## A couple of myths I’ve heard
• **Git can’t do binary files**. I suppose that technically this is true, but the [git-lfs extension handles this](http://git-lfs.github.com/) and is fairly mature at this point.
• **Git can’t lock files.** [git-lfs locks files quite easily](https://github.com/git-lfs/git-lfs/wiki/File-Locking) 

You should try to use both in this class, so that you can knowledgeably compare them. If you’re using Perforce for your group project, make sure to use Git for at least one of your assignments, and vice-versa. If you need a personal depot created on WPI’s Perforce server for an assignment just send me a DM on the course Slack.

## Using Perforce + Unreal
WPI has a Perforce server that you may use for your group projects and/or individual assignments. The details about logins and accounts for this server will be given in the course Slack, since they can’t be disclosed in a public document. In addition to this tutorial, you can also take a look at [the Perforce tutorial created by Epic](https://docs.unrealengine.com/en-US/Engine/Basics/SourceControl/Perforce/index.html) for more information.

1. Once you have your login information and the URL:port for the server, the next step is [downloading the Perforce software](https://www.perforce.com/downloads). You will mainly be using the [P4V visual client](https://www.perforce.com/downloads/helix-visual-client-p4v) but the command line tools and the merge tool are also useful to have.

Perforce organizes files into **depots** (not repos). A depot is a location on a central server where the “master” version of each file is held and versioned. A **workspace** is where your local copies of the files in a depot are held… this is where your project will go on your personal computer. Note that this distinction is a very different model than Git, in which there are simply copies of repositories that point to each other in different ways.

2. Open the P4V application and choose Connection > Open Connection from the main menubar. In the resulting pop-up, enter the url:port of the server and your user name. If you don’t already have a workspace created, make a new one now from the same menu in a directory of your choosing. Hit OK. In the main P4V view you should now be able to see both your Depot(s) and your Workspace(s). 

3. Make your new Unreal project (preferred) or move your existing Unreal project (headaches are coming your way) inside of your new workspace. YOUR PROJECT FOLDER SHOULD MATCH THE NAME OF YOUR DEPOT ON THE WPI PERFORCE SERVER.

4. Choose File > Connect to Source Control from the main menubar (you can also access this from the main toolbar) . Choose “Perforce” as the Provider, and then enter the server IP address / port, your user name, and then select your workspace from the available options in the dropdown menu… this will automatically populate the textbox immediately above it.  Hit the “Accept Settings” button.

5. You should (briefly) see a notification that the connection was successful! 

6. If you are the first person in your group setting up your project, go back to the P4V app and find the project folder in your workspace. Select the Config, Content and Source folders, right-click and choose “Mark to Add”.  Click on any individual file in these directories (or one of the directories) and click the “Submit” button. Select all the files to submit to your depot, and then enter a “changelist description” (similar to a commit message in Git). Hit submit.

7. Your files should now be uploading to the depot! If you select the Depot view in P4V you should be able to see them. You can now check out files for editing by right-clicking on them and selecting the Check-Out option. If you don’t check a file out before editing it, you will receive a nasty warning that the file is “read only” when you try to save… make sure to remember to check files out! Alternatively, you can do a quick web search on how to setup your workspaces to not require this.

8. Click the “Get Latest” button with your workspace folder selected to sync with the depot . REMEMBER TO DO THIS BEFORE YOU CHECK OUT ANY FILES FOR EDITING, SO YOU CAN AVOID MERGE CONFLICTS.

9. If you create a new C++ file inside of Unreal, you’ll see that it is automatically marked for adding when you view the file inside of P4V. You can use also Unreal’s Perforce plugin to add binary files (like models and animations) and to submit your changes without having to jump back to P4V. 
