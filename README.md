# Splunk - SSH Dashboard

## Forewords and Feedback
This write-up reflects my thought process during this lab and reasoning for actions taken.

Feel free to share your feedback and opinion or reach out to me, and also point out mistakes I may have made. I am of the strong opinion that learning from others and our mistakes are where we often learn the most or get the rare gems of knowledge, thank you for taking the time reading through my project.

## Objective
We have been set the task to create a dashboard in Splunk using SSH telemetry which includes the following data:
-   Top Account Failed
-   Top Source IP
-   Number of Failed Attempts by User
-   Successful Logins
-   Heatmap for all External Activity

## Pre-requests
-   VM - _(WMware Workstation)_
-   Ubuntu Server - _(24.04.1 LTS)_
-   Splunk Enterprise _(9.3.1)_
-   Index file to use in Splunk _(5931 Events)_
-   Internet Browser

## Making sure we are using the correct index file for the lab
We are using the command **”index="mydfir-lab1”** to check we have **5931 events** as this will confirm we are working with the correct index file.

![Events](https://github.com/user-attachments/assets/0bcbbd9c-20f2-4210-add7-409f5f639e64)

## Building our first query
We start out by looking for failed logins by using the following keyword **failed**, our query will look like this: **index=mydfir-lab1** failed and we get **284 events**.

![failed](https://github.com/user-attachments/assets/8b5f0c8e-c3f5-46c4-aa90-9e2cc3a5610e)

Our next step is to narrow down our search so we get better data out for our dashboard, there we want to include the fields **user, message, source IP** and **source port**.

![user](https://github.com/user-attachments/assets/9bf89b68-6e6b-490c-8545-fcc2c0fc1835)

Following selecting the fields we get the following output and our query now looks like this: **index=mydfir-lab1 failed host=linuxvm source="auth.log" sourcetype=linux_auth_logs |top user**

![auth log](https://github.com/user-attachments/assets/f2a557be-1a6a-4eac-904b-a24205ba8477)

## Creating our Dashboard
From the above result we see that the highest count of failed login attempts is for the user root, we wish to use this value in our dashboard. We do this though the **Visualization** tab and from the next window we pick **Single Value** and save it through the **Save As** Option and pick **New Dashboard**.

![Single Value](https://github.com/user-attachments/assets/53eede5a-69a9-464d-8099-4da83c9121da)

![Save As](https://github.com/user-attachments/assets/c0941bde-edc7-4cd5-9dc4-f05482cb9e3b)

In the next window we fill out details for the new dashboard:
-   **Dashboard Title:** SSH Activity (We enter a title for the dashboard)
-   **Description**: (We can enter a description for the dashboard and what it displays)
-   **Permissions**: (We leave it as Private, if we had the permission and working in a group we could have changed this to **Shared**)
-   **Classic Dashboards:** (We use the classic builder)
-   **Panel Title:** Top Failed Account (The title tells the viewer which account has the highest failed logins)
-   **Visualization Type:** Single Value

![Dashboard](https://github.com/user-attachments/assets/8ccba299-120c-4efc-b8f6-5bdf850cc364)

We have now a dashboard called **SSH Activity**. We click the **cross** as we want to add additional queries related to the SSH Dashboard.

![Cross](https://github.com/user-attachments/assets/a0333097-b8de-4511-a980-59a7896c1819)

## Adding more queries - Source IP
Next we want to add the top source IP who is attempting to login. We remove the **user** from top and exchange it with **src_ip**. New query is: **index=mydfir-lab1 failed host=linuxvm source="auth.log" sourcetype=linux_auth_logs | top src_ip**

![ip scr](https://github.com/user-attachments/assets/4cc6c19f-67f2-497b-84c5-48982c55f160)

We save and add it to our previously created dashboard **SSH Activity** and name the panel **Top Source IP**

![Top Source Ip](https://github.com/user-attachments/assets/731fcdfd-5ad5-4451-9458-b4349bdd4655)

## Adding more queries - Failed Logins by Users
We also wish to add a panel which displays failed login attempts by users. We do this by the query: **index=mydfir-lab1 failed host=linuxvm source="auth.log" sourcetype=linux_auth_logs | stats count by user | sort -count**

![Failed Logins](https://github.com/user-attachments/assets/4239262a-ed64-4304-baa3-e5d7e2023f57)

We save and add this to our existing **SSH Dashboard** and name it **Failed Attempts by User**

![Failed Attempts](https://github.com/user-attachments/assets/9a2f1813-50da-48d3-98fe-ad0184e4ea4f)

## Adding more queries - Successful Logins with Country
Next panel we create is for successful logins, we do this by the following query: **index=mydfir-lab1 host=linuxvm source="auth.log" sourcetype=linux_auth_logs**

![Successful](https://github.com/user-attachments/assets/b49ec03d-1e43-41a7-9535-540da86ee87e)

From the above result we get 919 events, this is a bit high and can give us a false true indicator of successful logins. We can narrow it down and get a better result by looking over the available field names, in our case we have a look at **msg** and herein we find the **Accepted password** which we find as a good indicator as it shows 6 events. Our query for this one is: **index=mydfir-lab1 host=linuxvm source="auth.log" sourcetype=linux_auth_logs msg="Accepted password”**

![Accepted pw](https://github.com/user-attachments/assets/640c184b-70e7-42a2-80d6-25ce05828011)

Additionally we would like to see from which country the successful logins are coming from. This will give us a quick indicator of the location of the user and if there is a compromised account etc. We add the stats command with the fields _time, user and source IP to our query. Our query is: **index=mydfir-lab1 host=linuxvm source="auth.log" sourcetype=linux_auth_logs msg="Accepted password" |iplocation src_ip |stats count by _time,user,Country,src_ip**

![iplocation](https://github.com/user-attachments/assets/5dcbe306-b0b0-41aa-ac29-81d4d7149605)

Having a quick look at the above result we see the logins are happening from Greenland and Canada. We add and save the panel to our Dashboard.

![User](https://github.com/user-attachments/assets/afb46067-5da9-4c13-95ac-c0bd83862f4c)

## Adding more queries - Creating a Heatmap
We have got a lot of data and numbers from our dashboard, this can be overwhelming so visualizing this through a heatmap can be useful, and can be useful for a faster overview or for decision making.

We build our heatmap by removing parts from our existing query, our new one is: **index=mydfir-lab1 host=linuxvm | iplocation src_ip | stats count by Country**

![Country](https://github.com/user-attachments/assets/d7b3f218-9ea9-4253-ad2c-bcd486bfd1a4)

Next step we use the **geom** command which takes two arguments, one called **allFeatures**, we give this value of **True**, the second is called featureField which we give **Country**. These stats command will count the feature ID and rename geom’s featureIField to Country. Our query is:
**index=mydfir-lab1 host=linuxvm |iplocation src_ip |stats count by Country |geom geo_countries allFeatures=True featureIdField=Country**

![geom](https://github.com/user-attachments/assets/27d6e576-661f-49c5-b7ea-2aca3be08445)

We create the heatmap by clicking on the tab Visualization and then clicking on the Map icon.

![Geo](https://github.com/user-attachments/assets/617c914d-0924-412b-bb6f-084f71fcaf09)

This will give us a map but it will not filter and give each login and country their own color.

![Canada](https://github.com/user-attachments/assets/c75795c0-7668-43f8-9fe9-69357e3eb3d6)

We get this through **Format - Colors - Categorical**

![Color](https://github.com/user-attachments/assets/9bc54f20-9c00-4ec6-ada9-acee439d3883)

![Colors](https://github.com/user-attachments/assets/790d2e99-2cbd-4d41-8a09-2df7574ab58b)

Lastly we save the heatmap to our existing dashboard.

![Heatmap](https://github.com/user-attachments/assets/07bf91e5-f5e3-4525-94a1-522f2d9514db)

Now we are done and do not wish to add more panels to our dashboard we click **View Dashboard**.

![View Dashboard](https://github.com/user-attachments/assets/956e87eb-e3eb-4741-b658-5e20ad77aaa8)

## Final Dashboard
Now we have all queries in our dashboard and we can now edit and format it to our taste, and how we best want the data presented and to more readable. To edit the layout we use the Edit button in the top right corner.

![Edit](https://github.com/user-attachments/assets/33261c49-f196-48d6-be5e-9b4079cf1a69)

![DashboardReady](https://github.com/user-attachments/assets/9f8d80aa-a6f5-460b-b5f3-66936308f8e5)

## Changelog
