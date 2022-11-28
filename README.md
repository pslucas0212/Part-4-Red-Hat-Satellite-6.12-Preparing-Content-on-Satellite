# Part 4: Red Hat Satellite 6.12 - Content on Satellite

[Tutorial Menu](https://github.com/pslucas0212/RedHat-Satellite-6.12-VM-Provisioning-to-vSphere-Tutorial)

In the section we will configure Satellite to manage content for our RHEL environment.  We will enable RHEL repositories on Satellite and define a RHEL lifecycle.  

### Adding Software Repositories to Satellite. 

We will be using and enalbing RHEL 8 and 9 content for this tutorial.

Login to the Satellite console and on the side menu navigate to Content -> Red Hat Repositories. 

![Content -> Red Hat Repositories](/images/sat15.png)

We will first search for RHEL 8 repositories.  Enter RHEL 8 x86_64 in the Available search field and then click the Search button. Now toggle the Recommended Repositories switch to On.  

![Red Hat Repositories Search](/images/sat16.png)

You will now see a smaller set of repositories.  We will be enabling three RHEL 8 Repositories:

Name | Repository
---- | ----------
Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs) | rhel-8-for-x86_64-appstream-rpms
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | rhel-8-for-x86_64-baseos-rpms
Red Hat Satellite Tools 6.9 for RHEL 8 x86_64 (RPMs) | satellite-tools-6.9-for-rhel-8-x86_64-rpms

![Red Hat Repositories Search Results RHEL 8](/images/sat17.png)

To enable a repository, click the twisty icon to the left of the repository name and then click the blue plus icon. If you hover your mouse point over the blue plus icon you will see a pop up text that says enable.  It is recommended to enable the version release of a RHEL repository and not the point release.  We chose the version release of the RHEL repository as it contains all errata from GA until that release is no longer supported.

![Expand Twisty](/images/sat18.png)

After you click the blue plus sign, you will see the selected repository is now in the right column titled Enabled Repositories.

![Enabled Repositories](/images/sat19.png)

Repeat the steps above to enable the other RHEL 8 repositories.  When you are finished, the Satellite console should look like the screenshot below.

![All RHEL 8 Enabled Repositories](/images/sat20.png)

We will now enable some RHEL 7 repositories.  Enter RHEL 7 in the Available search field, click the Search button and then toggle Recommended Repositories switch to On.  

![Red Hat Repositories Search Results RHEL 7](/images/sat21.png)

See the following table for the RHEL 7 repositories we will enable.  Follow the steps above to enable each repository. 

Name | Repository
---- | ----------
Red Hat Enterprise Linux 7 Server - Extras (RPMs) | rhel-7-server-extras-rpms
Red Hat Enterprise Linux 7 Server - Optional (RPMs) | rhel-7-server-optional-rpms
Red Hat Enterprise Linux 7 Server (RPMs) | rhel-7-server-rpms
Red Hat Satellite Maintenance 6 (for RHEL 7 Server) (RPMs) | rhel-7-server-satellite-maintenance-6-rpms
Red Hat Satellite Tools 6.9 (for RHEL 7 Server) (RPMs) | rhel-7-server-satellite-tools-6.9-rpms

We have chosen the content we need to manage for our RHEL environment.  Now we need to synch that content to Satellite.  We will "manually" synch the content.  You can create a synch plan, but we won't be covering synch plans in this tutorial.  

On the side menu click Content -> Synch Status

![Content -> Synch Status](/images/sat22.png)

On the Synch Status page click on the Expand All and Select All links.  And click the Synchronize Now button.

![Synch Status Screen](/images/sat23.png)

TThe Synch Status screen will now show the progress of synching these repositories from the Red Hat Content Delivery Network to Satellite.  Since this is the first time you are synching content, it will take a bit of time to complete.  You will likely want to look at creating a synch plan to schedule synchronizing content during times where your network traffic is lower.  When a repository has completed synching, you will see a message next to the repo that says Synching Complete.  Note: For the purposes of this lab, you may want to only synch content for RHEL 8.

![Synch Status Screen in action](/images/sat24.png)

### Creating Content Lifecycles in Satellite
After your content has completed synching, we will create a content lifecycle.  Content lifecycles give you the ability to match RHEL errata to RHEL servers running in a particular environment to match your SDLC.  You may have simple or complex lifecycles for your RHEL servers, and Satellite gives you the ability to easily create and manage RHEL server lifecycles.   In the following section you will  use the command line to create the lifecycle environment in Satellite.

Lets create our first lifecyce environment and link it to the Operations Department
```
# hammer lifecycle-environment create --description le-ops-rhel8-prem-server --prior Library --name le-ops-rhel8-prem-server --organization "Operations Department"
Environment created.
```
You can list the lifecycle environments with the following command.
```
# hammer lifecyce-environment list
```  

If you want to only list information for a particular organization add the --organization <organziation name> to the command.  
  
Next we will add a content view to the lifecycle environment.  Content view allows to control the specific content made available to environments
```
# hammer content-view create --description cv-rhel8-prem-server --name cv-rhel8-prem-server --organization "Operations Department"
Content view created.
```

 We want to add the repositories to the content view.  For this we need the repository ID.  The following command provides you with a "shorter" view of the repositories listing only the repository ID and name
 

 ```
# hammer repository list --fields THIN --organization-label operations
---|-----------------------------------------------------------------
ID | NAME                                                            
---|-----------------------------------------------------------------
5  | Red Hat Enterprise Linux 7 Server - Extras RPMs x86_64          
6  | Red Hat Enterprise Linux 7 Server - Optional RPMs x86_64 7Server
7  | Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server           
2  | Red Hat Enterprise Linux 8 for x86_64 - AppStream RPMs 8        
3  | Red Hat Enterprise Linux 8 for x86_64 - BaseOS RPMs 8           
8  | Red Hat Satellite Maintenance 6 for RHEL 7 Server RPMs x86_64   
9  | Red Hat Satellite Tools 6.9 for RHEL 7 Server RPMs x86_64       
4  | Red Hat Satellite Tools 6.9 for RHEL 8 x86_64 RPMs              
---|-----------------------------------------------------------------
```

In this example, for the RHEL 8 content view we need repository IDs 2, 3 and 4.  Your ID list may be different.
```
# hammer content-view update --repository-ids 2,3,4 --name "cv-rhel8-prem-server" --organization "Operations Department"
Content view updated.
```
  
Next we will publish the repositories to the library.  This will take a few minutes while the content is being published to the content view.
```
# hammer content-view publish --name "cv-rhel8-prem-server" --organization "Operations Department" --async
Content view is being published with task c050e764-25da-43b2-8a23-9122af5a9120.
```
We can jump back to the Satellite console to view the content being published.  On the left side menu chose Content -> Content View.
  
![Content -> Content View](/images/sat25.png)
  
On the Content Views screen click the link Content View name.
  
![Content View Name](/images/sat26.png)
  
You will now see the content being published to the Content View.
![Content View Publishing](/images/sat27.png)

Let's promote the content view from the Library to the le-ops-rhel8-prem-server lifecycle environment from the command line.
```
# hammer content-view version promote \
--content-view "cv-rhel8-prem-server" \
--to-lifecycle-environment "le-ops-rhel8-prem-server" \
--organization "Operations Department" \
--async
Content view is being promoted with task bdf2dba3-c4dd-4a66-8e82-a9fbd71b4298.
```
When the update has completed, you will see two environments listed in the Satellite console under Content -> Content Views. In the Content Views page in the Environments section you will see Library and le-ops-rhel8-prem-server listed.
  
Let's go back to the Satellite console to see which life cycle environments have the cv-rhel8-prem-server content view.  Make sure you have Operations Department for your organization and moline for your location.  
  
If you are not in the Content view, click Content -> Content View on the side menu.  
![Content -> Content View](/images/sat28.png)
  
On the Content Views page, click cv-rhel8-prem-server view link.  
![Content Views click cv-rhel8-prem-server](/images/sat29.png)
  
Observe on the cv-rhel8-prem-server page in the Environments column, you will see two environments: Library and le-ops-rhel8-prem-server listed.  
![le-ops-rhel8-prem-server observe Environments](/images/sat30.png)  
  
All the steps above of course can be completed through the Red Hat Satellite console following the paths outlined with the screen images.  
  
Finally we will create an activation key for registering our RHEL instances with Satellite and ultimately RHSM.  At the command line we will create an activation key with the following command.  
```
  # hammer activation-key create \
  --content-view "cv-rhel8-prem-server" \
  --lifecycle-environment "le-ops-rhel8-prem-server" \
  --name "ak-ops-rhel8-prem-server" \
  --organization "Operations Department"
  Activation key created.
  ```  
In the console we can view the Activation Key on the Red Hat Satellite console by navigating to Content -> Content Views on the side menu.  
![Content -> Activation Keys](/images/sat31.png)  

On the Activation Keys page click the link for ak-ops-rhel8-prem-server activation key.  
![ak-ops-rhel8-prem-server link](/images/sat32.png)  
  
Notice that ak-ops-rhel8-prem-server activation key is assigned to le-ops-rhel8-prem-server lifecycle environment.  
![ale-ops-rhel8-prem-server page](/images/sat33.png)
  
## References  
[Installing Satellite Server from a Connected Network](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.9/html/installing_satellite_server_from_a_connected_network/index)   
[Simple Content Access](https://access.redhat.com/articles/simple-content-access)  
[Provisioning VMWare using userdata via Satellite 6.3-6.6](https://access.redhat.com/blogs/1169563/posts/3640721)  
[Understanding Red Hat Content Delivery Network Repositories and their usage with Satellite 6](https://access.redhat.com/articles/1586183)
