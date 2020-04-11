## Version Control for Media-Rich Systems
Note: this tutorial assumes that you have some experience using version control software, whether that's Git, SVN, Mercurial, or some other system. If that’s not the case, please contact me so we can cover some basics together. 

While versioning text is a (somewhat) solved problem, versioning binary files is much more difficult. There is no way to effectively diff or merge binary files, which means we’re left with completely replacing binary files from one version to the next. A few different source control management (SCM aka Version Control Systems or VCS) systems can help us with the process, and manage binary files between teams of users. 

The two we’ll discuss in this class are Git (with the git-lfs extension) and Perforce. There are good reasons for choosing both, and the list below is by no means to be considered comprehensive. If this influences your decision at all, I will be more useful as a resource for Git than Perforce… long live open-source software!

### Pros for Git
• **It’s open source and free to use**. Anyone can setup their own git server, although companies like GitHub, BitBucket, and Assembla have made this largely unnecessary.  
• Widely used across multiple industries and academic fields.  
• Distributed repositories by default.  

### Pros for Perforce (aka HelixCore, because, marketing I guess)
• Widely used at AAA studios. Because Perforce got a jump on handling binary files in a reasonable way, it quickly gained mindshare in game studios.  
• Centralized repository by default. Helps to keep everything in sync. You can also create a personal server for local commits.  
• Game Editor (Unreal, Unity etc.) support is typically a bit more robust than it is for Git.  

Here’s [a very biased (and there’s some actual misinformation in there) comparison from Perforce](https://www.perforce.com/blog/vcs/git-vs-perforce-how-choose-and-when-use-both)

## A couple of myths I’ve heard
• **Git can’t do binary files**. I suppose that technically this is true, but the [git-lfs extension handles this](http://git-lfs.github.com/) and is fairly mature and easy to install/use at this point.  
• **Git can’t lock files.** [git-lfs locks files quite easily](https://github.com/git-lfs/git-lfs/wiki/File-Locking)  

You should try to use both in this class, so that you can knowledgeably compare them. If you’re using Perforce for your group project, make sure to use Git for at least one of your assignments, and vice-versa. If you need a personal depot created on WPI’s Perforce server for an assignment just send me a DM on the course Slack with the proposed name of the depot.

## Using Perforce + Unreal
WPI has a Perforce server that you may use for your group projects and/or individual assignments. The details about logins and accounts for this server will be given in the course Slack, since they can’t be disclosed in a public document. In addition to this tutorial, you can also take a look at [the Perforce tutorial created by Epic](https://docs.unrealengine.com/en-US/Engine/Basics/SourceControl/Perforce/index.html) for more information.

1. Once you have your login information and the URL:port for the server, the next step is [downloading the Perforce software](https://www.perforce.com/downloads). You will mainly be using the [P4V visual client](https://www.perforce.com/downloads/helix-visual-client-p4v) but the command line tools and the merge tool are also useful to have.

Perforce organizes files into **depots** (not repos). A depot is a location on a central server where the “master” version of each file is held and versioned. A **workspace** is where your local copies of the files in a depot are held… this is where your project will go on your personal computer. Note that this distinction is a very different model than Git, in which there are simply copies of repositories that point to each other in different ways.

2. Open the P4V application and choose Connection > Open Connection from the main menubar. In the resulting pop-up, enter the url:port of the server and your user name. If you don’t already have a workspace created, make a new one now from the same menu in a directory of your choosing. Hit OK.

In the main P4V view you should now be able to see both your Depot(s) and your Workspace(s). 

3. Make your new Unreal project (preferred) or move your existing Unreal project (headaches are coming your way) inside of your new workspace. YOUR PROJECT FOLDER SHOULD MATCH THE NAME OF YOUR DEPOT ON THE WPI PERFORCE SERVER.

4. Choose File > Connect to Source Control from the main menubar (you can also access this from the main toolbar) . Choose “Perforce” as the Provider, and then enter the server IP address / port, your user name, and then select your workspace from the available options in the dropdown menu… this will automatically populate the textbox immediately above it.  Hit the “Accept Settings” button.

5. You should (briefly) see a notification that the connection was successful! 

6. If you are the first person in your group setting up your project, go back to the P4V app and find the project folder in your workspace. Select the Config, Content and Source folders, right-click and choose “Mark to Add”.  Click on any individual file in these directories (or one of the directories) and click the “Submit” button. Select all the files to submit to your depot, and then enter a “changelist description” (similar to a commit message in Git). Hit submit.

7. Your files should now be uploading to the depot! If you select the Depot view in P4V you should be able to see them. You can now check out files for editing by right-clicking on them and selecting the Check-Out option. If you don’t check a file out before editing it, you will receive a nasty warning that the file is “read only” when you try to save… make sure to remember to check files out! Alternatively, you can do a quick web search on how to setup your workspaces to not require this.

8. Click the “Get Latest” button with your workspace folder selected to sync with the depot . REMEMBER TO DO THIS BEFORE YOU CHECK OUT ANY FILES FOR EDITING, SO YOU CAN AVOID MERGE CONFLICTS.

9. If you create a new C++ file inside of Unreal, you’ll see that it is automatically marked for adding when you view the file inside of P4V. You can use also Unreal’s Perforce plugin to add binary files (like models and animations) and to submit your changes without having to jump back to P4V. 

## Using Git + Unreal
Although we don’t have a Git server for you to use [there](GitHub.com]) [are](bitbucket.org) [plenty](assembla.com) of [free options](gitlab.comm). The process will be roughly the same no matter which service you use. I am a [long-time Github user](https://github.com/charlieroberts) so I’ll be assuming use of GitHub for this tutorial. Personally I use git from the command line, but [GitHub Desktop](desktop.github.com) is also a very nice front end which I’ll use here and in the video version of this tutorial. 

1. Install [`git-lfs`](https://git-lfs.github.com). The `lfs` stands for “large file storage” and is an extension to git with improved support for binary files, including [locking them](https://github.com/git-lfs/git-lfs/wiki/File-Locking).

2. Go ahead and create your new Unreal project… if you played around with Perforce, make sure you don’t place it in your Perforce workspace by default.

3. Choose File > Connect to Source Control from the main menubar (you can also access this from the main toolbar). Select “Git” as the provider. If you’ve [configured your username](https://help.github.com/en/github/using-git/setting-your-username-in-git) and [your email](https://help.github.com/en/articles/setting-your-commit-email-address) for Git those should be automatically entered for you; you can also do this in the preferences of GitHub Desktop. If you know the origin server for you repo you can enter it now, otherwise you can ignore it; we’ll push to a new GitHub repo later in the tutorial. Go ahead and leave the rest of the options at their defaults. Note that by default, Unreal doesn’t currently support `git-lfs` but there is a [Unreal plugin that supports this](https://github.com/SRombauts/UE4GitPlugin). We’ll manually add in `lfs` support in a bit instead of using that plugin but feel free to try it out and let us know how it goes in the course Slack.

4. Go ahead and open up GitHub Desktop. Choose File > Add Local Repository and then navigate to your project folder. Select View > Show History to see a list of what Git has already committed to your project’s repo.

5. In Unreal, make a new C++ class inheriting from Actor. Right click on it in the content browser. When you do this Unreal will automatically run `git add` on the new files, but it will **not** make a commit for this. You can then choose Source Control > Submit to Source Control to make a commit, or perform the commit on the command line or a GUI Git client like GitHub Desktop.

6.  Right click on your new C++ class, and then choose Create Blueprint class based on… from the contextual menu. You’ll see the resulting Blueprint has a ? icon on it to show  that it has **not** been added to the repo… save your project to fix this… you should now see a +

7. After ensuring you [have git-lfs installed](https://git-lfs.github.com) for your user account, enter your project directory from the command line and then run the following two lines:
```
git lfs track "*.ucasset"
git add .gitattributes
```

8. … and now you’re set to use version control binary files! Remember that for you binary files (like all blueprints files, plus models / animations etc.) it’s a good idea to [lock them](https://github.com/git-lfs/git-lfs/wiki/File-Locking) before you begin to edit them. With regular C++ files you should be able to do a merge if necessary, so there’s no lock available in Git. In Perforce, any file can be locked.

## Locking
What does it mean to “lock” the file? In Git, it means you can’t push a new version of the file to the “origin” server. In Perforce, it means that you can’t check out the file to make edits. You can set a given file to forbid concurrent edits in Perforce [using the Change Filetype command in P4V or from the command line](https://community.perforce.com/s/article/3114).  With the command line, it is also super simple to use Git to [lock them](https://github.com/git-lfs/git-lfs/wiki/File-Locking) ; unfortunately, [it doesn’t appear that GitHub Desktop will support locks any time soon](https://github.com/desktop/desktop/issues/8419).

Either way, it’s important to use locks when you’re editing the binary file. A simple alternative is to post to your Slack channel when you’ve started editing these files and then post again when you’ve finished, but it’s easy to forget one or both of these steps and then get in trouble. Locks are worth learning to do right.

It’d be great if someday Blueprints could be simple text files instead of binary so they could be properly versioned… but we’d still have the same problem with textures / models / animations etc. Get used to locking your files ASAP!
