


Introduction
0:04
hello everyone welcome to the latest project on code with DOI today we're going to build a full stack social media
0:10
app using the latest Expo outer and super base in this project you will learn how to make beautiful and
0:17
responsive UI we will Implement superbase odd so user can login using their email and password you'll be able
0:24
to post images and videos as well you can create likes comments and you can even share the post with your friends
0:31
there are so many other features built into this app so let me quickly show you a demo and then we will start building
0:37
this app so as you can see the app is fully responsive on IOS and Android so let me just quickly log out and when we
0:45
first open the app it will show a welcome page like this and if we click on getting started it will move us to
0:51
the sign up page where we can fill all the information and create an account and this is the login page so let me
0:57
just quickly use my account and log to the application once we click the login
1:04
button you will see this loading animation and it will move us to the home screen now we don't want to save the password and on the home screen you
1:11
will see all the post from every user and if you scroll all the way down you will see a loading animation because we
1:18
have implemented pagination so as you scroll down it will fetch more and more
1:23
posts and if there is no more post then it won't call the AP again I'll show you how you can implement this then you can
1:30
like any post using this hard icon you can create comments and you can even share this post so if I click on this
1:36
icon I will be able to share this image and I can also save this image into the
1:41
gallery then if I click on this comment icon it's going to open the details model where I can see all the comments I
1:48
can also open this detail model using this three dots icon it's going to open the same model where I can see the list
1:54
of the comments and I can even create comments so let's create this comment
2:00
and as soon as this commment is added you will see uh there's a new notification on this second device
2:06
because I'm logged in as John know on this device so I'll show you how you can implement this real time updates from
2:13
superas so this is the homepage now let me show you the user profile section here is the profile where you can see
2:20
the user details and if we click on this edit icon you'll be able to edit all the information so let's change the profile
2:28
image uh let's use the this one and uh let's also change the bio to
2:35
code with Nomi and let's hit update this is going to update the user profile data so image
2:42
is updated and bio is updated in this section you will see only the user post
2:48
and we have also implemented bination here as well so you can also upload
2:53
videos in the post so this is a video post from this account so you can move
2:59
forward this is is a fully working player for our videos you can make it full screen uh you can move forward move
3:07
back so this is how the videos are working now let me show you how you can create a post so if you click on this
3:13
plus icon we move to the create post screen where you see this fully working
3:18
Rich Text Editor and we have a lot of styling features for the post body so
3:24
let's add the text as colorful and now I can make it board or italic
3:30
then I can move it to uh Center or to the right side or to the left side then
3:36
if you scroll we have other features as well so I can make it heading one to make it big or heading four and these
3:43
are not all of the features from this Library I'll show you how you can Implement all of the features and uh
3:49
this will help us to modify the styles for our post body we can also add media to this post so if I click on the image
3:56
icon it will show all the images from our Gallery I can choose the image it will show the preview we can remove this
4:03
using this icon and if we click on the video icon it will show only the videos from our gallery and I can even uh trim
4:11
this video and then select this video for our post so let's choose this video
4:16
now we can see the video and this is a fully working player so you can check the video before posting so let's remove
4:23
this and add the image for this post uh let's choose the flaw and hit post
4:31
and when the post is created it will instantly appear on both of the devices and that's because we're using the real
4:37
time changes from super base I'll show you how you can Implement those then if I open the post details on this second
4:45
device uh let's create a comment to see how the notifications are working so let's say nice and add this
4:52
comment the comment is added and also a new notification is added on the first
4:58
device so if I click on this not notification icon it will show all the notifications for this user these are
5:03
the old notifications for testing and if I click on this uh new notification it will open the post details and highlight
5:10
the comment that was created this is very useful because this way user can know which comment was related to the
5:16
notification so let's create another comment morning and let's click
5:22
Send the comment is added and so the notification item so let's click on this
5:29
new item and it will highlight the new comment that was created so this is very helpful next let's see how we can delete
5:37
or update any post so if user opens his own post he'll be able to see two icons
5:42
at the top edit and delete and if we click on the delete icon uh we will show the same page but with the post data so
5:49
let's change this text to morning and let's also remove the
5:55
styling and let's update and when this is updated it will be
6:02
updated on both of the devices it's updated on IOS and Android from both accounts and uh uh you won't see the
6:10
comments uh count here on this home feed because we are not using realtime updates here but I'll show you how you
6:16
can do that but for now simply if we just reload the application the comments count will be updated now let's see how
6:24
we can delete comments and the post these icons will only be shown to the post owners and and for delete comment
6:30
only the post owner and the comment owner can delete them so let me open another post to show you what I
6:38
mean so this is my comment and I can delete this but if I open the same post
6:44
from Jon Snow's account he won't be able to delete this because only Sarah and I
6:49
can delete this comment now let's see how we can delete the
6:54
comment so if we click on and the delete comment icon it will confirm us
7:00
then if we delete it will be removed and now uh let's see what happens if we open the notification related to that comment
7:07
so if I click on this uh it won't highlight anymore because the comment wasn't there and if we delete the post
7:14
we won't be able to see the post details so let's delete this and as soon as we delete this it's gone from both of the
7:20
devices because of the real time changes now let's open the notification item and
7:26
now it will say post not found because we just deleted it so this is how the real time changes are working and how
7:33
the notifications are working so this is just a brief demo of all the features you can view all the post upload images
7:40
videos see the profile section update profile data uh create comments likes
7:46
and share the post if you want to upgrade your react native skills then this is a video for you you can find the
7:52
source code link into the video description also you can find so many other great apps like this one on my
7:59
channel so make sure to like the video and subscribe the channel for more content like this so let's start
8:07
building this app so first we're going to create an export outer application so first you'll
Login & SignUp UI
8:14
need to go to export. d/ router installation or if you go to desort
8:20
dodev in the sidebar you will see this installation option just click on it and you'll find this page so there is this
8:27
manual installation guide but we're going to stick to the quick start so let's just copy this
8:32
command and we're going to open the
8:38
terminal uh let's move to the desktop and paste this command
8:43
here and for the name of this application we're going to use super social app CU we're using super base as
8:50
well now this is going to download all the template files for us and in the meantime let me show you something so
8:59
we're going to use eraser. I to track the progress of our application or to draw any mockup of the screen and in the
9:07
sidebar I've added all the steps that we're going to do while building this application and in the canvas we can
9:13
draw anything so for the setup uh we have uh created the export order app so
9:18
I'll just mark this as done now we're going to perform all of these steps now the application is downloaded and uh we
9:25
can close this terminal I've already opened the project into vs code so I'll just drag it here and make it full
9:33
screen let's also add our simulator here so I've already open the simulator so
9:39
let's adjust them both in the single window like this uh let me make it
9:47
smaller now let's try to run this application and see what it looks like so let's open the terminal and run the
9:54
command npm run iOS now this is is going to build this
10:00
project for us and open in Expo go and it's building okay the app is running and we
10:08
have two tabs at the bottom explore and home screen we're using the latest expore outer SDK so if you see in the
10:15
package. Jon we using 51 SDK and if you go into the app folder you will see all
10:21
of the code for these tabs explore and home but we're going to delete all of these files because we going to start
10:28
from scratch so let's delete all of these files and in the assets uh in the
10:34
components we will delete all of these files as well but we'll keep the components
10:40
folder and in the constants we don't need this colors file so let's delete this as
10:47
well and hooks um we don't really need the hooks so let's just remove
10:53
them okay so uh now we have a very clean slate to start with so let's create our
10:59
first component index. GSX uh don't worry about this error just create a functional component inside this and
11:07
let's save this now we're going to have to refresh reload our application and we
11:12
will see our first component as you can see the index file now if you want to learn more about export outer I will
11:18
leave the link of a detailed video about export outer into the video description
11:23
so you can check it out so now we're going to perform all of these steps starting with the theme setup so let's
11:30
go back to our application and create a theme file into the constants and I'll
11:36
just copy and paste the theme configuration here this is an object containing the colors fonts and the
11:41
radius that we will use throughout the development of this application so we
11:46
have the basic colors primary dark gray we have the uh fonts which we use as the
11:52
font fight and the radius is I've defined all of this into this file because if we need to change anything
11:59
we'll just change this here now let's create another file index.js and uh
12:05
let's just leave it empty for now but we'll use this later we will also use some helpers to design our application
12:11
so let's create helpers folder and inside this let's create a common.js file for our common
12:18
helpers now to make our app responsive we're going to give some of the Styles as percentages and for that we will use
12:26
the device height and device width to calculate a percentage value for the style and we can get the device height
12:33
and width from the dimensions object from react native so this will uh get us
12:39
the height and the width of the window which is actually the height and width of the current device now we will create
12:45
a function HP that stands for the height percentage and this will receive a percentage value now using that
12:52
percentage value we will calculate uh value using the height and width of the current device so percentage multiplied
12:59
by the device height divided by 100 that will give us the height percentage and
13:06
we'll just copy this one for the width percentage so let's copy this and call
13:12
this WP as width percentage and we'll just change the device height to device
13:17
width now we can use this to make our design responsive now let's create a component inside the components folder
13:24
as screen wrapper this is going to wrap all of our screen and we're going to give a padding top to this component so
13:30
that our content don't conflict with the notch area so this will receive children
13:36
props as and the background so let's remove this and here we will display all
13:42
the children's and let's add a style to this let's give this a flex one so that it
13:49
takes full space and for the pading we will use a safe area inserts so that we
13:55
have a padding on all of the devices so let's use the top value from use safe
14:00
area inserts and first we will check if uh the Top Value is greater than one that
14:07
means uh this device has a notch and in that case we will give the padding as top plus 5 pixels otherwise we will have
14:15
a 30 pixels of pedding now let's use this in our Styles object as the pedding
14:24
top and let's also add a background color as BG
14:30
which we are passing to this component so let's use it here and everything's good to go so let's save this now we're
14:37
going to use this but let's review what we have done so far so setup theme is done helpers are done and Screen wrapper
14:44
is done next we're going to design our welcome screen so let's create a welcome screen into our app
14:52
folder like this and let's create a functional component and I'm able to use
14:58
this short codes to create a component quickly and I'll show you how you can do that so if you head over to extensions
15:06
you will need to install uh this extension es7 Plus react Redux Snippets
15:12
and you will also need to install this simple react snippers extension so then you'll be able to use these snippers to
15:19
create your functional or Clause components quickly so now let's use our screen wrapper
15:25
here and let's save this but uh we are not able to see the welcome screen yet
15:31
so let's go to the index screen and here let's create a button that will move us
15:36
through the welcome screen don't worry about this our first page will always be the welcome page but uh the reason we
15:43
are using this index page because later we will use this as unloading when we implement the authentication so uh just
15:51
go with the flow for now so when we press on this button we will use our router to push to the welcome screen
15:58
like this uh save this and if we go to welcome we are moving to welcome screen but uh we have this pedding at the top
16:07
which I think we haven't defined into our screen wrapper but I think it's
16:12
coming from the header from our layout because we are in a stack layout so let's create a layout file into the app
16:19
folder and uh let's create a functional component and here we will need to
16:25
return the stack layout so let's return a stack stack layout and save this now
16:31
here you can see we have the header but if we hide this header maybe this padding will go away so screen options
16:38
and add header shown as false now the header is gone but we
16:45
don't see the index page so we will need to use our screen wrapper here as well
16:50
so let's use screen wrapper and let's save this now if we go
16:56
to welcome page we don't see that padding and the header is gone now let's
17:01
start working on our uh welcome screen but first we're going to create a mockup of our welcome screens so this is the
17:09
welcome screen and let me just add a label here like this now here first we
17:14
will have an image of the welcome screen on the top so let's say this is the
17:20
welcome screen image and at the bottom we will have our app name or app logo we don't have a
17:27
logo so we'll just have an app app name uh it will be big like this and then
17:33
after this we will have a punch line for our application this will be small and this will show after the app name now at
17:41
the bottom we will have a get started button so let's create a button and this
17:46
will be get started now I know this looks very ugly but we'll make this
17:52
beautiful using the Styles so let's start designing this and before that let
17:58
me just import all of the assets into the images folder so let me just copy
18:03
and paste them here so here we have the default user image which will show in
18:09
case user don't have an image and the welcome image which is shown on the welcome screen so let's go to our
18:16
welcome screen let's remove this and we're going to give a background of white to our screen wrapper so let's use
18:23
BG equals white then inside this component we are going to use use a component called
18:30
status bar from Expo outer to make the fonts into the status bar as dark like
18:36
this now uh let's use another view uh which will be the container so let's
18:42
give this a style as styles. container which we will create in a
18:49
second like this now uh we already have a stylesheet here so let's create
18:55
stylesheet as container let's give this a style of Flex one align items as
19:03
Center and justify content as space
19:09
around background color as white and the pading horizontal as uh
19:16
now for the padding horizontal we're going to use the wp method that we created earlier so uh this will Auto
19:23
Import um let's go to our helpers oh we forgot to export it so let's export the
19:30
both of these functions now this will Auto Import from this file now let's use
19:35
WP and now we can import this and let's give this WP of four that will mean WID
19:41
percentage of four but we are not able to see it because the background is
19:46
white so if I change this to red and change the padding to margin horizontal
19:51
then we will see a slight margin around the container and we can change this using a bigger value and we will have a
19:58
big margin so this way you can use these values as percentages to give the style
20:04
as responsive style so we will use these now inside this container we will use
20:09
the welcome image first so let's import the image component from react
20:15
native then for the style we'll use styles. welcome image which we will
20:21
Define later and for the source uh we will use the welcome image that we just
20:28
copied from asset SL imagesw welcome.
20:33
PNG and let's save this we can already see the image but we will add the Styles
20:39
so let's also add resize mode as contain now let's add the styles for
20:45
this image as the welcome image uh we will give this a height and width using
20:51
the HP and WP method so for the height HP uh which will be the height
20:56
percentage of 30 and this will take the 30% of the overall height of the device
21:04
and width will be 100 uh which will mean the full height full width of the device and align self
21:11
to the center so that the image shows into the center like this and the image
21:17
is looking good now and now after this image we're going to show the app title and the punch line so let's add a
21:23
comment here first uh as the title and let's create a view and give
21:29
this a style of Gap of 20 this will be the gap between the title and the punch
21:35
line and let's add a text component for our app name and let's add the Styles as
21:42
styles. title and the title of our app is link up and let's copy this text component
21:50
for the punch line as well so let's change the styles to punchline and remove the title and add our punchline
21:57
which is where every thought finds a
22:02
home finds a home and every image tells a
22:10
story like this now if I save this we can see the title and the punch line but
22:15
uh we need to add the Styles so let's add the style for the title now we're going to use the color from our theme
22:22
object so let's import the theme object so we can use all of these different
22:28
colors from our theme as the primary dark or text color so let's use the
22:34
color text for this title and we can use the HP method to
22:39
make the font size responsive so let's use HP of four that will main 4% of the
22:46
total height let's also align the text into the
22:53
center and for the font weight we already have the font weights into our theme object so let's go to theme. fonts
23:02
uh if we go into the fonts we have the medium semibold and the Bold so let's use extra
23:08
bold and now we have an extra bold uh title of our app now let's add the
23:14
styles for the punch line and uh we will have the text line
23:20
into Center adding horizontal of uh WP
23:27
10 and the font size of uh HP 1.7 we
23:32
need to make it smaller and the color will be the same text color for that we used for the
23:40
title let's save this now we have the title and the punch line styled now at
23:46
the bottom we're going to have a footer area where we will display a get started button and some login text so let's
23:53
create a view give this a style of footer
23:58
and here we will have a custom button that will say get started so we'll create this custom button so we can use
24:04
it after so let's create a component inside this folder
24:10
button and let's create a functional component now let's save this and import
24:17
this here uh we will have to pause some properties later but for now let's leave
24:23
it empty so now we can see the button so let's go inside this and this button will receive uh some properties as a
24:32
button style then the text style uh the title for this
24:39
button and an on press method uh which is an empty function for
24:46
now but uh we will provide the function later so a loading by default it will be
24:52
false and a property has Shadow if we want to have Shadow for this buttons so
24:58
by default this will be true now uh let's uh change this View to a
25:05
pressable and the onpress method will be the on press function that will pass to
25:11
this and for the style we will have multiple styles for this button so first
25:17
we will provide the styles from stylesheet so Styles or button we will
25:23
provide this here these will be the default Styles then we we will uh provide the button
25:30
style that we pass to this from parent component and uh then we will also have
25:35
some Shadow Styles so let's create an object here Shadow Styles and this fun this uh this object
25:43
will only be applied if the has Shadow property is true so if has Shadow is
25:49
true then we'll provide Shadow Styles like this and same for the text so by
25:54
default we will have the text style from uh the Styles sheet in this component so
26:00
styles. text and the text stle from the parent component like this and this button will
26:08
display the title so we can pause the title from the parent component so let's
26:13
pause the title here as getting started and if I save this you can see
26:20
uh we have the getting started title now let's add a button style as well and give this a margin horizontal of
26:31
wp3 and we can also pause and onpress function so let's leave it empty for now
26:39
so uh now we're going to have to style this button then we'll come to this function again so for the button style
26:46
let's add the background color from our theme object as the primary
26:53
color like this then uh for the height of this button let's use our HP function
26:59
to give this a height of 6.6% then justify content as Center
27:06
align items as Center and then border curve as
27:14
continuous and let's use the Border radius from our theme object as
27:19
[Music] theme uh we need to import the theme theme. radius. xcl and these are already
27:27
defined in to our theme object so we can use these multiple radiuses now uh let's
27:32
save this and uh this looks weird but we'll fix this later so now let's add
27:38
the style for the text so text style and let's give this a responsive
27:45
font size of HP
27:50
2.5 then color will be white and the font fight we already have
27:59
the font weights into theme object so let's use the font weight of bold and
28:05
let's save this now the reason why it's showing this because it doesn't have a full width so let's go into our welcome
28:13
screen uh not this one so welcome screen and we need to provide the full wids to
28:20
our footer so let's give the styles for the footer as let's add a gap of 30
28:25
pixels then width will be be 100% like this okay so the button is looking good
28:33
now this is our custom button that we can use later so uh we're going to use this multiple times so uh the only thing
28:41
left is the shadow styles for this button so let's add the shadow Styles let's do Shadow color as the dark color
28:48
from our theme then Shadow offset which will be an object width
28:56
will be zero and height will be 10 pixels like this and Shadow opacity as
29:08
0.2 Shadow radius will be eight and then the elevation will be
29:15
four for Android devices let's save this and now we can see the shadow behind the
29:20
button and if I make the has Shadow property false we won't see this Shadow so we can control the Shadow by using
29:27
this has Shadow property from the parent component as well so this component is almost complete we are just going to
29:34
show the loading State using this loading property so if this loading is true we're going to make a condition
29:40
here if loading is true we're going to return a view here that will display the
29:47
activity indicator so let's create a view and give this a style as the Styles start
29:53
button actually we need to give this uh the Styles start button and the button
29:58
style because we want to take the same space as the button is taking so Styles
30:04
start button and the button Styles then the background color will be white like this and inside this we're going to
30:11
display the activity indicator so let's create another component for our loading
30:16
state so uh into the components folder let's create a new
30:23
file loading. GSX and let's create a functional component
30:29
uh now this function will receive the size and the color property for this loading indicator so by default the size
30:36
will be large and the color will be the primary color from our theme this will
30:41
be the default color but but we can change these properties from the parent component so uh let's give this the
30:49
style as justify content into the center and item center line item Center then in
30:58
inside uh this we going to return an activity indicator from react native
31:04
give this a size as the size and the color will be color from the
31:12
props let's close it and save it now we can use this loading component
31:19
uh in the button component so let's move to button and here import the
31:26
loading and let's say save this now uh we need to make the loading true to see
31:32
to see this so let's make it true and now we can see the loading
31:37
state so this is how the loading state will show and if this is false we will display the button so this component is
31:45
complete now we can move back to the welcome screen and add the login text at the bottom of this button so we'll
31:52
change this later now let's add a view after this button and give this a start
31:58
of um bottom text
32:04
container like this and inside this we're going to have a text component
32:10
with the style of login text and this will say already have an
32:18
account if yes then user will click on the login button so uh after this let's
32:24
add a pressable for the login button and inside this we will have a text
32:31
component that will say login so for the style of this component let's use styles. login
32:40
text like this and this will say login and if I save this we can see the
32:48
text but we need to style it so let's go to our stylesheet and add bottom text
32:54
container and give this a flex direction of row justify item
33:00
Center and align item Center and let's save this so uh we need
33:08
to add a gap between these text so let's add a gap of five pixels okay it looks good now let's add the style for the
33:15
login text and let's give it text line
33:21
Center and uh text color as the text color from our theme
33:28
and then the font size using our HP function as
33:34
1.6 and let's save this okay so the text looks good but we need to make the login
33:40
text as the button so that user know it's a button so that's why I added an
33:45
array of the Styles here so let's change the color as the primary dark color from
33:50
our theme and let's also make it semi board so font weight as theme do font
33:58
do semi board and let's save this now this looks more like a button so that user can
34:05
click it so this welcome screen is complete and next we're going to move to the sign up and login screens using the
34:12
getting started and the login button so first let's create those components let's create the login
34:18
screen and a functional component let's change the name and save it let's create
34:26
the sign up page a functional component and uh let's
34:31
change the name and save it now we're going to use a router to
34:38
move to these screens inside the welcome screen so let's use the used router Hook
34:43
from export outer and when we click on this button
34:48
we have this onpress method this will push a new route as the signup so we'll
34:54
move to sign up screen and for this login button we will use onpress method
34:59
and this will push us to the login screen that we just created so let's save this and see if
35:07
this is working so go to welcome screen and if I hit getting started we move to
35:12
sign up and if I hit on the login we will move to the login screen
35:21
okay so let's move to our steps and we have completed the custom button and the welcome screen next we're going to work
35:27
on login and sign up screen and I'm going to show you how you can use SVG icons using a very cool Library so we're
35:35
going to perform all of these steps back button icons so let's create a mockup of
35:41
the login screen so let's say this is our login screen let's add the label as
35:48
login screen first we will display uh text at the top welcome back this will be
35:55
displayed here then we will have have some fields for the email so this will
36:01
be email and let's copy this for the password then we will have a button at
36:08
the bottom that will say login so let's just copy this one and paste it here
36:14
login and this will match the theme so let's give it a green color so this will
36:20
be our login screen and same will be used for the signup screen so let's just
36:26
copy this the only thing we will add here is uh we will have a username here
36:32
so this will say get started and this will be the
36:38
username um actually this uh we won't display the username here so let's move
36:44
these at the bottom and at the top we will display the username because we also want to get the
36:50
username and uh then email and then password then the login button sorry uh
36:56
this will say sign up like this so uh let's also change this
37:01
label sign up screen okay so both of these screens looks very simple but they will look
37:08
good once we finished with all of these steps so first we're going to create a
37:13
back button inside the login screen so let's move to the login screen and first
37:19
we need to add a screen wrapper so that our content is away from the notch area
37:25
and for the back button we're going to use the icon so let me show you the library that we're going to use and it's
37:31
called the huge icons it's a paid library but we can use the stroke icons for free so let me show you how you can
37:39
use the icons as an SVG into react native so first I'll search for the home
37:45
icons and let's press enter and this will show show all the
37:51
icons that we can use so let's use this one and uh if I press on this icon at
37:58
the bottom this will expand the page and there you can find the react native code for showing this icon as an SVG so for
38:06
this to work we need to install this react native SVG Library so let me just install this first let's open the
38:14
terminal and npm install react native
38:19
SVG and this will install the library we will also restart our server so let's
38:26
write NB on iOS again so if we have any error we will
38:32
fix this right now so the app is running now let me just uh refresh the
38:38
app okay so we don't see any errors that's a good sign so we're going to use
38:43
this back icon in the login component but let me show you how we can actually use the component so this is the react
38:50
native code for this home icon so I'll just copy all of this code and uh let's
38:56
create a folder into my assets as the
39:02
icons and I'll create a new file in here home.
39:07
GSX and I'll copy and paste all of this code so now let's change the component
39:13
name to home and you can see we are passing some props to this component and if you
39:21
scroll you will see this stroke width which is uh fixed but we don't want to
39:26
have a stroke width as 1.5 we want to be able to control it using from our parent
39:32
component so I will provide it as props do stroke width so we can pass it from
39:38
the parent component and it will be updated now uh it's very simple to use
39:43
this component so let's go into our login screen and here I'll just import
39:48
the home icon from our assets and if I close it and save it uh
39:56
there we go we can see the home icon that we just copied and we can provide some of the properties here like stroke
40:03
width as let's say two this will increase the stroke width as you can see
40:10
and we can also change the color so if I go into home we can see the color
40:16
property which is black so we can change this color property because we are passing the props from the pent
40:22
component and then spreading all the props here so I'll be able to change the color from from here as well so let's
40:28
use the primary color from our theme object and this will update the color of
40:34
the icon and we can use other icons so let's use rose from our theme and this
40:41
has updated the color so this is how you can use the icons from the huge icons Pro so the process is very simple you
40:47
just search the icons in our case we going to use the back arrow and if I search for Arrow I can use all of these
40:54
icons if I search for let's say image I can use these icons for media so let's
41:00
say I want to use this icon I will expand this and use uh I can copy all of
41:05
this code then I can move to our application and create a new file here
41:11
uh same way we created home and we can change the stroke width if you want to be able to control it so this is the
41:19
process now I'm going to create an index file here because I want to generalize
41:25
using these icons so I just have to import one file and use all the icons so
41:31
this component will be icon so let's change the name to Icon like this and we will have an
41:38
array of all the icons here so uh this the the key will be the name of the icon
41:44
and then it will uh return the icon component from the same folder and we
41:50
can use other icons like Arrow left and this will have the arrow left icon from
41:56
this directory we will add these icons later but uh we don't have this for now
42:01
so let's just use home this component will receive some icon props starting with the icon name and some other props
42:10
that will include height width or stroke width so uh first we need to get this
42:15
component so icon component using the name on the icons
42:20
array uh this will be name like this and once we get the icon component we can
42:26
return this here with all these uh props from the parent component uh we will
42:31
have uh some fixed height and width if there is no size property from the props
42:36
so uh first we need to give the size property from the props if it doesn't
42:41
exist then we will use 24 height and same goes for the width if we have the
42:47
size property then this size otherwise 24 then for the stroke width uh we will
42:55
use the stroke width from our prop and if it doesn't exist the default stroke
43:00
width will be 1.9 like this and for the color uh we
43:06
will use the light text color from our theme but we can obviously change this
43:12
because we will be spreading all the props here so in case if we need to change any of these properties we can
43:18
just pause it from the parent component and this will be updated so this will be icon as
43:24
well uh now uh we can just import import this icon component and give this a name
43:29
and this will show the icon so let's go into the login component we no longer need to import the home icon and let's
43:36
remove this now we can import the icon from the icons folder and just give it a
43:42
name as home and save this and this will display the same home icon again and we
43:47
can pause the other properties and these will work so color red and this is updated so I've created all of the files
43:55
for the icons that we're going to use go into this application so I'll just copy and paste them into the icons
44:02
folder um yes replace so these are all the icon files
44:08
that I just copied from the huge icons that uh the way I just showed you guys before so let's go into the index
44:16
component and let's copy the icons array here as well for all the icons so these
44:22
are all the icons now let me just copy and paste the import statements for all
44:27
of these icons so let's remove all of these and paste it here okay we have all the icons and
44:35
let's save this uh Styles sheet doesn't exist I
44:40
think we don't need this so let's just remove it okay so now we'll be able to
44:46
use all of these icons from this icon component so let's go into login and uh
44:52
let's change the name to mail now this is showing the mail icon and if I change this to delete and this will say the
44:58
delete delete icon and arrow left Arrow left icon so now we can use all of these
45:05
icons just using the icon component and uh uh let's go into our
45:13
steps uh we have completed the icons part now we're going to work on the back button so the back button will go
45:20
actually on the top left corner so uh let's see if we can use the icon Arrow
45:28
so there is the back button so this will be displayed here at the top left corner
45:34
on both of these screens so now let's create a back button component into our
45:39
application into the components folder let's create a new file back button. jsx
45:47
and let's create a functional component let's save this now let's first uh
45:52
import this into the login let's remove this and first we're going to use the status bar component to change the style
46:00
do for our status bar text then uh let's create a
46:05
view and give this a style of styles do
46:10
container and inside this we're going to have our back button
46:15
component uh let's close this and save this so we can see the back button now
46:21
let's go into the back button and start designing first uh this view will be be
46:27
a pressable so that we can press on this back button and then inside this we're going to use the icon so let's import
46:34
the icon and the name will be Arrow left then stroke width will be
46:44
2.5 and the size uh we will actually pass the size from the parent component
46:49
but by default it will be 26 but we can change it from the parent component so
46:54
let's use size and for the color let's use the text color from our theme object
47:00
like this and let's save this now it will be able to see the icons there is
47:06
this icon now let's apply the back function on this pressable so on press
47:12
uh we actually need to pass the router to this component for that to work so let's move to the login uh component and
47:20
let's create a router here use router from Expo router then let's pause the
47:25
router to this back button like this and if we go into the
47:32
back button we can use the router so we can use the router. back function to go back to the previous route now let's add
47:40
the styles for this button so let's use the styles. button
47:45
style now we will Define this style into our stylesheet so
47:50
button and first we will give it an align self of flex start
47:58
then the padding of five pixels and Border radius from our theme
48:04
as the small then we're going to have a background color so we will have a
48:11
transparent black color so let's use R CBA color black color with
48:17
0.07 opacity and let's save this okay so the button is looking good now if I
48:23
click on this button we should go back to the previous route and and let's see if it's working okay the back button is
48:30
working so next we're going to work on our login screen so let's move to our
48:36
login component now here I can Define all of
48:42
the Styles one by one and I can show you how you can style each and every component but I don't want to do that uh
48:49
because this will take a lot of time to complete this whole application so I'll just copy and paste the styles for this
48:55
component and when I use them I will show you what styles I've used for any
49:00
component so let me just copy and P the paste the whole style sheet
49:05
here like this and let's save this I will use all of these styles for this
49:10
component but I will show you each and every style so uh we need to import WP
49:17
and HP methods and then the theme object as
49:22
well that's why we seeing this error so for the container as you can see we have
49:28
Flex Gap and the padding horizontal of wp5 you can use these now we're going to
49:33
have a welcome text so let's add a comment welcome and then a view and
49:39
inside this let's have a text container give this a style of Welcome text and
49:46
then this will say hey welcome back so let's just copy this and change
49:52
this to welcome back like this now let's save this and we can see the
49:59
welcome back text and the styles for this are font size font weight as this
50:04
and the color as the text color from our theme so you can use this so uh next
50:10
let's add uh we're going to add the input so let's add a form text then
50:15
let's add a view and give this a style of styles. form which we already have in
50:21
our theme so if I click on the form style you will see we only have a gap of
50:27
25 pixels for each input item so let's create a text component and uh let's
50:34
give it a style of uh let's use font size of HP
50:39
1.5 and for the color uh let's use the theme colors of
50:46
text and inside this this will say please log to
50:51
continue like this okay so now let's create the input UT component for our
50:57
email input but I want to use it as a custom input component because we're
51:03
going to use it multiple times and it will be better to use it later for other components so uh let's create a custom
51:10
input component inside the components
51:16
folder and let's create a functional component and this will receive some
51:21
props from the parent component so let's receive them and uh this will be the container
51:27
view so let's give this a style of styles. container let's use an error because we
51:33
can pause the container styles from the parent component as well so if we have the container styles from the parent
51:40
component we will use these like this and let's remove this
51:47
text and uh we can also pause the icon from the parent component so if we have the icon then we'll just simply display
51:54
the icon here and for the textt input let's import it from the react native
52:01
and let's give it a style of Flex one we going to pass the placeholder from the parent component but let's have the
52:07
placeholder text color as the text light color from our theme
52:14
object like this and the reference will also be paed from the parents component
52:20
if we need it for some reason so Props do input ref we will use this if we are passing
52:27
from the parent component then the rest of the prop props we will just spread them here in case if we pass any so
52:35
let's close this now let's design for this container Styles so let's specify
52:41
the container Styles here container and then uh we going to have the flex direction as row because we can
52:48
display the icon and the input so the height will be HP uh 7.2
52:57
and then the Align items Center justify content into the center as well and
53:06
let's choose the Border width of uh let's say 0.4 and the Border color will be from
53:12
our theme so let's use a text color then the Border radius will be a dou XL from
53:19
our theme radius doou XL border curve will be
53:29
continuous and bedding horizontal of 18 pixels then finally we're going to have
53:36
a gap of 12 pixels because we are displaying the icon and the input so we
53:41
need to have a gap between them now let's move back to the login component and here let's use the input component
53:48
that we just created so first we're going to use the icon for this input so let's use the
53:54
icon component uh make sure you import the icon from the assets slash icon uh
54:00
so we have this here then uh let's add the name as mail icon then the size will
54:07
be 26 pixels and for the stroke width we will use
54:14
1.6 like this and let's close it then let's add the place folder for this
54:20
input as uh this will say enter your email so let's add it
54:28
here like this and next we will have on change text function uh this will
54:33
trigger every time user types A word so this will give us a value so let's leave it empty for now let's close it and save
54:42
it and there you can see we have our custom input with the icon now we're
54:47
going to save the email or the password into a reference so let's create
54:52
reference for the email and password you can use the state as well but the thing is the state updates every time user
55:00
types even a letter so that's why I'm using a reference but you can use the state so let's use the password
55:10
reference now let's also create a state for the loading so whenever the loading is true we will show a loading instead
55:17
of the button so by default this will be false like
55:22
this now let's change the email reference current value whenever we got
55:27
the value from the email input like this and let's just copy this for the
55:32
password input and let's change this to enter your
55:40
password so now now if I type anything I can see whatever I type but I don't want
55:46
to do that I want to hide it so for that we need to use a property as secure text
55:52
entry on this input so if I add it here
55:57
we won't be able to see uh now whatever we type just like this just like how password Fields work now let's change
56:04
the icon to lock icon there we go so these icons are coming from the input uh
56:12
component that we created and you can either copy them from huge icons or you can find them into the source code I
56:18
will leave the link of the source code into the video description now uh we going to have a forget password text
56:26
after these inputs so let's create a text component here and I'll give this a
56:31
style of styles. forgot password we already have this into our
56:40
stylesheet and this will say forgot
56:46
password with a question mark like this so if I go into the F password this says
56:53
text line on the right font weight and the color so you can use these Styles now let's
56:59
add a button here and this will be the login button so let's import our custom
57:05
button from the components and give this a title of
57:12
login and then we going to have the loading state which we already defined
57:17
here and then an on press method which will uh trigger the on submit
57:24
function now let's create cre this on submit
57:34
function okay so you can see the button and uh let's see if everything is
57:41
okay okay so we're going to have to change the background color so let's change the background color from our
57:47
screen wrapper let's give it a white color like this and uh after this uh
57:53
button we're going to have a footer area where we will display if the user don't have an account then he can just uh go
57:59
to sign up so let's add footer area let's give this a style of
58:06
footer and in the footer Styles we have all of these Styles so you can just uh
58:11
copy these from here then inside the footer we will have a text component
58:17
with a style of styles. footer text and this will say I don't have an
58:24
account then just simply go to sign up page so for the sign up we will uh make
58:31
this as a button so let's create a pressable and inside this pressable
58:38
we'll have a text component with the same style as the footer text so this will just say sign
58:45
up we're going to use the same Styles as the footer text but we're going to make sure it looks like a button as well so
58:53
uh we will add more Styles like the color so let's use the color as the
58:59
primary dark color from our theme then the font weight will be semibold so
59:05
let's use theme. fonts. semibold like this and let's save this
59:13
now it looks like a button so our component is almost complete uh let's
59:18
let's justest if the loading is working so uh if we change the loading state to True uh we should see the loading
59:25
indicator and yes we do and let's change it back to false now we're going to do the
59:31
validation here so if I click on the login we should validate login and the password and this should be password
59:38
reference and if we have both the email and the password only then we will call
59:43
the API so let's check if we don't have the email reference. current value or if
59:49
we don't have the password reference. current value then uh we will use an
59:55
alert to say to the user that please uh fill all the
1:00:02
fields like this and then we will just simply return
1:00:08
from here but if we have both of the values then we are good to go and we will call the API but not now so let's
1:00:16
see if this is working so if I click on it we see the error and we if we only
1:00:21
have the email we will still see this error and if we have the password too uh we don't see this error so we are good
1:00:27
to go so with this our login component is complete and we can just copy this
1:00:32
for the sign up and we change a few of the things so let's copy this whole
1:00:38
code and we'll go to sign up and paste this here and let's save
1:00:43
this um uh I don't know why but the application just scratched so let's
1:00:49
start it again okay it's working fine let's go to
1:00:56
the login component and this is working so let's change a few of the things for
1:01:02
the signup component and let's change the component name to sign
1:01:07
up let's also change the name when we export this component in
1:01:13
here and let's save this um we're getting a warning
1:01:20
here navigation state was from the oh this is nothing we can just
1:01:26
ignore it so uh let's go to the sign up
1:01:31
page and it's still the same as the login so let's change uh some settings
1:01:37
we already in the sign up page so this will say sign
1:01:42
up like this and uh we will also add a name a name input so let's create a
1:01:49
reference for the name input and then uh this will say
1:01:55
let's get started so let's change the welcome back to get started like
1:02:03
this and then uh this line will say uh please enter the
1:02:10
details to create a new account like this now we need to create
1:02:19
a new input for the name so let's copy the email and change the icon to user
1:02:24
this will show the user icon and let's change the placeholder to enter your
1:02:29
name like this and this will be name reference so now user can able to enter
1:02:36
the email password and the name we don't need this forgot password here so let's remove this and this button will say
1:02:44
sign up like this and this will say uh
1:02:49
already have an account then please press the login button
1:02:56
so for this we will need to change the sign up to login now we need to be able to move to
1:03:04
these screens so we need to go into the login screen and first implement the routing here as well so when we press on
1:03:11
this button using the onpress function we need to move to the signup screen so router. push to the sign up and let's
1:03:20
copy this the same for the signup screen and here we will just go to the login
1:03:26
page let's copy this here and change this to login like this so if we click
1:03:32
on the login we move to login page and if we click on the sign up we move to sign up page let's see if the validation
1:03:38
is working on the sign up so if I click on the sign up button yes the validation
1:03:44
is working and with this we just completed the UI for the login and the sign up
1:03:49
pages so now let's see what we have done so far we have implemented the custom
Supabase Auth
1:03:56
back button so this is done and the validation is done next step is a superas setup so now we're going to add
1:04:02
super base into our login and sign up pages so let me show you the super
1:04:07
website so this is uh the project I was working with before but you will need to
1:04:14
go to super.com and once you create an account you will be redirected to your dashboard
1:04:19
and uh there you can create a new project so let's go to dashboard and create a new project for our
1:04:26
application uh you will have to choose your organization so mine is say Nan so I'll click on this then for the project
1:04:34
name uh I'll choose super social
1:04:40
app like this then type a strong password for your
1:04:46
database then choose the region which is obviously uh in your case closest to
1:04:52
your audience and then create the project so once the project is created
1:04:57
you can see your uh service Ro Anon key and the project API key and in the sidebar you can see all the features
1:05:03
like database All Storage real time so we're going to use all of these in a
1:05:09
later but uh now let me show you the guide to setup authentication in react
1:05:14
native so this is a very comprehensive guide uh this shows you how you can create uh the application create the
1:05:21
login components then create then Implement authentication we already we have created the application so we just
1:05:27
need to install in these dependencies this is the superbase client Library so I'll just select
1:05:35
these and copy them and let me just close the server
1:05:41
and install all of these libraries in
1:05:49
here okay so these are installing now uh we already have our login comp component
1:05:56
and the sign of component so we don't need to create it uh again and uh we
1:06:01
just need to create a Super base. DS file in our lib folder and then have all
1:06:06
the configuration there so let's create lib folder and then superas
1:06:13
dogs then let's just copy this client configuration so this code basically
1:06:19
creates a superas client which we can use to uh Implement authentication so
1:06:24
I'll just copy this and paste this here and the libraries are installed so let's run our
1:06:32
application okay so now we're going to get the subas URL and the Anon key for
1:06:38
these variables so that we can create a client and let me show you how you can get these values so go into your
1:06:44
superbase project and uh there are multiple ways you can get these API keys so once you create the project and
1:06:52
you'll be redirected to API key section and there you can find the project key and the Anon key here but if you lost
1:06:59
that page you can go into the sidebar into the project settings and under the API section you
1:07:06
will find the API keys so uh you can find the project URL and the Anon key
1:07:12
there is one more way to get the API Keys just go to your home section and click on this connect button and then
1:07:19
click on the mobile framework this will show you the project URL and the Anon key so you can copy this and paste this
1:07:25
into your application so let's copy the project URL and let's copy this into our
1:07:33
constants so we already have constants index file so let me just export a
1:07:38
constant here super Bas URL and paste the key here then let's
1:07:44
also export the Anon key so superbas Anon
1:07:50
key and let's copy it from our project this is the Anon key
1:07:56
paste it here and save this now we can import these AP Keys into this file so we don't
1:08:04
really need to have these variables so let's just remove them and directly import them and use them here so super B
1:08:11
URL and the super base Anon key like this okay so this will create a client
1:08:18
for us and in the bottom here this is uh just an event listener for superbas to
1:08:23
autore refresh the token so uh let's go to our guide and we have created the subas client next step is to
1:08:30
create the login component which we already have then uh we need to use the signin with password function and send
1:08:38
the email and the password to login with superbas but first we need to sign up the user so we will use the sign up
1:08:44
method and we will pass the email and password and this will return us a session for that user if the user is
1:08:51
created successfully so we already have the so let's go into application and
1:08:57
implement this sign up so let's go to um application is running so let's just
1:09:03
refresh the app okay the app is running and let's go
1:09:09
to our sign up page and here using the superbas client we will implement the
1:09:16
sign up functionality so let's go to sign up page and uh uh here in this
1:09:21
function we already have the variables so we will just trim the name email and the password so if there are any blank
1:09:28
spaces it will just remove them so trim the name let's copy this two more times
1:09:34
for the email and for the password as
1:09:42
well okay so this will get rid of the blank spaces then uh we going to start the loading because we're going to call
1:09:49
theas signup function so let's see how we can use this uh we can to call the signup
1:09:56
function with an object email and password then it will return a data and then error in the response so let's uh
1:10:04
get the data and the error
1:10:09
object and we need to use a wait superas do o so this is the client that we just
1:10:16
created we need to import this and then we can use the OD service and then the
1:10:21
sign up function there we go so uh uh this will be a function that will
1:10:27
receive email and password in an object like this so once this is created we need to
1:10:34
console log the data so let's console log the session and we can get the session inside the data so let's get the
1:10:45
session like this and we will show the session and
1:10:51
let's also console log the error if we have any
1:10:57
and in case we get any error we want to uh show an alert box to the user so we
1:11:02
will make a check if there is an error uh we will use the alert from direct
1:11:12
native and this will show sign up error as error. message and uh finally we need to stop
1:11:21
the loading so uh let's stop the loading here make it false and let's save this
1:11:27
now before we test everything we need to uh enable the email provider so if it's
1:11:33
not already enabled go into your authentication Service and in the sidebar you will see providers so let's
1:11:39
just refresh the page okay so in the sidebar you can see
1:11:44
the providers click on the provider and by default the email will be enabled but we need to turn off this uh confirm
1:11:52
email option because uh if we don't it will send a confirm email uh on the
1:11:57
email that we used to sign up and we need to perform an extra step I don't want to do that so uh we just need to
1:12:04
make turn it off and then let's try to register an
1:12:09
account and then hit sign up so we saw the loading and in the
1:12:14
console we see error as dull and we got the session data this has the access token that means uh our uh user is
1:12:22
logged in and it successfully register on the super base so there we can find
1:12:27
all the user data we have the email email verified false phone verified and
1:12:33
the user ID so uh we can see this user into the super base as well if you go
1:12:39
into the users uh you will see all the registered users here so uh there it is
1:12:44
the newly registered user So currently we can only see the users here but I
1:12:50
want to store all the users into a users table so let's go into table editor
1:12:55
and then create a new table and for the name let's use uh
1:13:00
users then superbase has Ena the role level security but I'll come to this later so ID uh will be The UU ID for
1:13:09
each user which will be randomly generated then we have the created add property which is a timestamp then let's
1:13:16
add more properties as name this will be a text property then uh let's add the image
1:13:26
this is also a text property then bio this will be a text column uh these
1:13:34
will be null uh by default but we will have an option to update all of these properties in the user profile section
1:13:41
so address and finally uh we will have a phone number property column and all of
1:13:48
these will be text properties and by default they will be null like this so
1:13:54
this is the users table and if you scroll to top you will see an option enabled Ro level security this is a
1:14:00
security protection from sub base which allows the certain users to perform
1:14:05
certain operations on this table so we're going to have to create some policies to be able to create or read
1:14:12
data from this table so uh let's click on this button add RLS policy this will
1:14:17
take us to the policy section and here we can create a policy for this table so
1:14:22
in the sidebar you can see we already have some predefined uh templates enable row access for users enable delete users
1:14:30
based on the user ID so we'll use this and uh we will modify this so that uh
1:14:36
all the users can perform all the uh operations but only for authenticated
1:14:41
users so uh I will select all of the actions so if I click on this and then
1:14:49
uh for the Target role I will use authenticated so this mean authenticated users will be able to to select insert
1:14:56
update or delete any data from this table so this policy is done so now we
1:15:02
can add data into the users table but because of this o policy that we just
1:15:07
created but for now you can see the table is empty so the way uh we want
1:15:12
this to work is when we register the user of using our application we want to
1:15:17
send some data that will also add him to the users table that we just created but
1:15:22
for now if we just simply register any user it won't appear instantly in the users table so let me show you what I
1:15:29
mean so let's create a new
1:15:35
account and hit sign up so the account is created so if I go
1:15:42
to uh authentication I will see two accounts but if I go to the users table I won't
1:15:48
see the new user here and that's because we have to create some kind of trigger that will uh create a user into the user
1:15:56
table so we have to follow this guide I found this very useful user management guide so this will provide us a way to
1:16:04
create a trigger that uh will create the user into users table every time a new
1:16:09
user registered so basically this creates a function and a trigger so this is a function body so this is a function
1:16:17
body that uh creates the user into users table and this is the trigger on o
1:16:22
schema so superbas has this o schema where every authenticated user is stored
1:16:28
so if we uh switch the publ schema with au schema here uh we will have this
1:16:35
users table here here is all the authenticated users so uh what we doing is basically we are adding an event on
1:16:42
this schema so every time there is an insert in all. users we want to run this
1:16:48
function so that it will store this user into our users table and we can send
1:16:54
this addition information in options. data and we'll be able to fetch this
1:16:59
data inside the function but you will have to modify this SQL statement because in your case the table name and
1:17:06
the column names can be different so I've done this I'll go to SQL editor and
1:17:11
copy and paste my SQL statement here so basically what this do is it it
1:17:19
will create a handle new user function which returns a trigger and this is the function body uh this will insert the
1:17:26
user into public. users with the ID and the name and we can get the name using the raw user metadata which we will
1:17:34
pause from uh the front end using the sign up options and then finally it will listen for the inserts in au. users so
1:17:42
uh let's run this and it successfully ran now we can see this trigger and the function into the database so if I go
1:17:50
into the functions here we can see uh we have a new handle new user function
1:17:55
which returns a trigger and we can see this trigger as well if I go into the
1:18:01
schema as Au so there we go we have a new trigger
1:18:07
which listens for the user table and after every insert it will create a new
1:18:12
user into users table which is in the public schema so uh let's delete all of
1:18:17
the existing users and we will try this and see if this works let's delete this
1:18:29
okay so now we don't have any users so let's just refresh our app go into sign
1:18:37
up and let's create a new user as Sayad
1:18:49
Nan so with the sign up now we need to pass this additional name data as well
1:18:55
and we can pause some options and the data to uh fetch this inside the function so we can provide option and
1:19:02
then data object inside this we can pause additional data as name so if I go
1:19:08
into the user management guide here you can see uh we can pause additional data
1:19:13
like this and we will be able to fetch this inside the function so if I go into
1:19:19
the SQL editor in here and here you can see we
1:19:24
are fetch fetching the data from the new object new object basically is the new user object and which contains all the
1:19:30
user data and we can fetch the name from raw user metadata so if I click on sign
1:19:37
up now we have create successfully created the user and uh if I see in the
1:19:43
super base let's go into authentication uh we see the new user here and now if I go into the table
1:19:50
editor users okay there we go we have the new user automatically inserted into
1:19:57
the users table we can see the user uu ID and the name that we posted uh from
1:20:03
the sign up function so this is how we can use the trigger and the function now
1:20:09
let's create one more user JN
1:20:18
snow and hit sign up so the user is created and we can see it's inserted
1:20:24
into into the users table and we can see this user into the authentication as well let me just refresh and we can see
1:20:31
both the users here so our signup functionality is working now uh we can
1:20:37
remove this console log and then move to the login component and then implement
1:20:43
the login functionality let's go into the on
1:20:50
submit function and here first we need to trim the email and the password so let's just copy them from the signup
1:20:57
component and paste them here now first we need to start the loading so set this loading to
1:21:05
true and then we will get the response from the sign in function and this will
1:21:11
contain an error uh we will import the superbas client and then
1:21:17
Au DOT sign in with
1:21:22
password and this will receive a object with the email and the password so let's
1:21:27
I use email password here and console log the error if we have
1:21:34
any we'll also check check if we got any error we just alert the user so let's
1:21:39
make a condition here if there is an error let's choose
1:21:45
alert and show the login error as error do message like this let's also uh F
1:21:52
make the loading false here after the API call now let's save this and test if
1:21:58
the login is working and if it's working we will see the error as null so let's
1:22:04
log to my [Music] account and hit
1:22:11
login okay so we see the error as null that means the login is working and if I
1:22:17
change the wrong password we will see this error invalid login credential so that means our login functionality is
1:22:23
working now uh we want to move to a home screen after we sign sign in or uh sign
1:22:30
up so let's create a new uh group of routes inside our app
1:22:36
folder let's call it Main and inside this let's create a home
1:22:41
component let's create a functional component and change the name to Capital
1:22:47
like this so now we need to move the user to home screen whenever he logs in
1:22:52
or registers and for that super base has this Au listener which triggers every time when a user logs in he registers or
1:23:00
when he signs out so this is the code snippet for using this trigger it's
1:23:05
called on All State change and this gives us an event and a session uh which
1:23:10
contains the user data and we can show this in all the components or we can redirect the user to home screen but
1:23:18
before we implement this we need to store the user into a context so that we can use the user data in all the
1:23:24
components so let's create an O
1:23:30
context and here first we will need to use a create context function from
1:23:37
react this will be our context and now uh we need to create a provider so let's
1:23:44
export this HS provider this will receive the children
1:23:50
components and uh here we will have a state for the user and sech user by
1:23:55
default it will be null and when it's null we will just move the user to welcome page but if it's not then we
1:24:02
will move the user to homepage then we have this set o function which will set
1:24:07
the session of the user and uh based on this session we will redirect the user
1:24:12
to home screen and then we will have another function set user data and I will
1:24:19
explain why I added this function but later this will just take the user data
1:24:25
and uh spread the data into the user State like
1:24:32
this so now let's provide all of these functions and the user state to all the
1:24:37
children components so o context do provider and the value will be all of
1:24:43
these functions so user State set o and set user
1:24:49
data now let's close this and render all the children here so this will wrap our
1:24:56
entire application so that we can use this user State uh in all the components
1:25:01
now let's recreate this use o hook this will uh use this
1:25:06
context and let's provide our o context like this so now we're going to use this
1:25:12
o provider inside our root layout so that uh we can use this user State into our entire application but before that
1:25:20
we need to change this layout to main layout because if you want to use the use o hook we need to wrap our entire
1:25:27
layout with uh this o context sorry o provider so let's create a new layout
1:25:33
and inside this uh first we will return our o
1:25:39
provider and that will wrap our entire stack which is the main
1:25:46
layout and uh now in in the main layout we'll be able to use the set or Set uh
1:25:52
user or the user state from our context so let's get the set o function from our
1:25:58
use Au hook use o like this now to be able to
1:26:06
use this we first going to implement the O listener which triggers every time user logs in and we already have that
1:26:13
here so we just need to copy this own o State change function uh the second one
1:26:19
so let's just copy this one and we will need to create a a use effect hook and
1:26:25
inside this we will register this let's just paste this
1:26:31
here so first we will need to import this superbas client from lib SL
1:26:36
superbas so this is the Au listener which will trigger every time user logs in or he registers or uh when he logs
1:26:44
out but in the session we will get a user value so let's just console log the session
1:26:50
user and see if we are getting the user data
1:26:57
so let's save this and uh you can see in the console the user session is
1:27:03
undefined because we are currently logged out but I want to uh make a condition here so that if we get the
1:27:10
session that means we need to go to the home screen so if we have the session then we need to do two things uh
1:27:17
navigate to home screen and then set the authentication uh from this session so
1:27:23
set all o and if we don't get this session then we need to set the O as null and we need
1:27:31
to move to the welcome screen like
1:27:37
this so uh before we implement this let's just if the session is working if
1:27:43
we log in so by default the session is uh session user is uh undefined and if I
1:27:48
try to log in let's see what happens
1:27:57
okay if I log in you will see uh we got the session user and it this has all the
1:28:03
user data from uh authentication so I believe we should be getting the user ID
1:28:10
uh we can also see the user metadata and we have the email and the name of the
1:28:15
user so this is the user ID which is a unique hash it's a uu ID that we stored
1:28:22
into the users table while we implemented the trigger and the function so instead of showing this session user
1:28:29
object we can just display the user ID like this and save this um not now so now we can see the
1:28:37
user ID so now let's implement the authentication and first we will need to
1:28:43
set the O as session. user and we need to move to the home
1:28:49
screen and for that we will need to use the router so let's create a router use
1:28:54
router Hook from Expo router so we will replace the current route so that user
1:29:01
cannot go back to this welcome page again so we'll just replace it with home screen and if there is no session that
1:29:07
means user just logged out or there is no session so we'll just set o as null
1:29:14
and replace the current route with the welcome okay um what's happening
1:29:21
now I think we made a mistake somewhere why it's started to move in a loop let's
1:29:30
just close this um everything looks fine but uh maybe it
1:29:36
was just a glitch from for superv I don't know so let's just uh try to run
1:29:42
this again and uh see if this
1:29:50
works okay so so still the same
1:29:57
error and it's going in a loop let's review what did we do
1:30:04
wrong oh oh okay so we need to add this empty array uh for the dependencies so
1:30:11
uh it was updating again and again and that's why it was moving to the home screen in a loop so let's see if this
1:30:19
works this time and now we move to home screen and
1:30:25
that's it so we can't see the home text properly so let's go to home and here we
1:30:30
need to import our screen wrapper like this and we can see the
1:30:37
home screen so let me explain how all of this is working when we open the
1:30:42
application this super Bas o listener triggered and we got the session and when we have the session we set the
1:30:49
session and move uh the user from current screen to the home screen and that's what happened now we don't have a
1:30:56
way to go back so let's create a button this will be our log out button
1:31:02
for now so title will be log out and on press uh we trigger on log out
1:31:09
function let's create this function here and uh first we will need to set o as
1:31:15
null so let's get our set o function from our use Au hook
1:31:25
like this and uh set o as null then we're going to call the sign out
1:31:30
function from super base so this will give us an error if there is any so
1:31:36
let's await for the super base do O then sign out
1:31:44
function and if there is any error we will check if we got any error we just alert the user that we got an error
1:31:52
signing out so this will say log out and signing error signing
1:32:00
out um let's change this to sign out and I save this now if I press on
1:32:07
this button uh we should go to the welcome page and it's working so
1:32:12
basically when we logged out this session got updated to undefined then we set Au as null then move to welcome page
1:32:19
so I just noticed we don't need this set o here because we already set setting o as null here so if I just refresh we are
1:32:27
still on the welcome page so we still shortly see this index page because this
1:32:33
is the uh first page of the application so if I just comment this session logic
1:32:38
we will still see this welcome page but I want to make this as a loading uh
1:32:43
component so uh whenever the session is being fed from the super base off
1:32:49
listener we will just show a loading State and then once the session is fetched uh we will move to move the user
1:32:55
to either welcome or the home screen so I'll just make everything into the center and then import our loading State
1:33:03
and save this now this shows a loading so if I uncomment the session logic here
1:33:11
I will move to welcome screen so even if I refresh we see the loading for a very
1:33:16
small period of time then we move to welcome or to the home screen so now with this our authentication system is
1:33:23
complete so let me check if everything's working so let me just log in with my
1:33:30
account okay so I think the password is
1:33:37
incorrect so when we login we get the updated session into the S listener and
1:33:44
we display the user ID from the session here and if I log out this will update
1:33:49
the session to undefined and we move back to welcome page and this is working now let me log in one more time to see
1:33:56
if we refresh the app or if we remove the application from recent apps will it
1:34:01
still persist the user so let's see if I refresh the app we are still on the home screen so
1:34:08
session is working and if I remove the app and open the Expo Go app
1:34:14
again we are still on the home screen and that's because superas uses async
1:34:19
local storage to persist the user session and whenever we open the application it triggers the OWN o stage
1:34:25
function uh with the updated uh session so that's why we are logged in so with
1:34:31
this we have done our superb setup users and Au trigger and Au context as well
1:34:37
now we're going to work on creating all the tables and the policies in the super base but before that let me show you one
1:34:44
more issue so uh if I console log the user data that we have into our context
1:34:51
so we can get the user from use o and if I console log this basically we got this user from the
1:34:59
session in the O State Lister and this does not include all the properties that
1:35:04
we defined into the users table this has the email the name and the subproperty
1:35:09
as the ID but uh we want all the information that we defined into the users table so if I go to users uh you
1:35:17
will see we have the name image bio email address and the phone number we need all of these properties to display
1:35:24
in the user profile so what we will do is when we get the session and move the user to home we will call a function
1:35:31
here that will fetch the updated user and then set set it into the user state
1:35:37
in our context so that way we'll get all the uh user fields from the users table
1:35:43
so let's create this function here and this will be an async function
1:35:49
it will receive the user from our session so now we need a way to get the
1:35:55
user from users table and for that we can go to the services folder uh which
1:36:00
we haven't created so let's create a Services folder and inside this let's create our
1:36:08
first service as the user service and let's export a
1:36:15
function const get user data this will be an async function which will receive
1:36:22
the user ID and we will get the user and we will return it from here so first
1:36:27
let's use try cach block and if we get any error we will console log the error as
1:36:35
got error and we will return a response as
1:36:41
success as false and a message with the error
1:36:47
message like this so now we're going to call the supervis API to fetch the data from user
1:36:53
us table and uh when we call this API this will get us a data and an error in
1:36:59
the response so let's use sub client then using the form uh form uh function
1:37:06
we need to provide the users table basically we are getting the data from users table so uh these functions are
1:37:13
very easy to use if you are an SQL Developer you'll be familiar you'll get familiar with this very easily but if
1:37:21
you're not here is the documentation for for getting the data uh you can use form
1:37:26
select you can uh select specific columns like name you can query the reference data so I'll provide this link
1:37:33
into the video description but for now we only need uh the user uh with the
1:37:38
user ID so what we can do is use the select function then this will select
1:37:44
the data from the users table and if you want to apply a condition we can use EQ
1:37:49
as the equal function this will check if the ID is equal to the user ID that we're passing then this will give us an
1:37:57
array of the data but we want only single item and we can use a single function to get the single object now uh
1:38:05
let's see if we have any error we will just return the error response from our
1:38:10
cach block then if we don't get any error uh we will return the success response so let's return success as
1:38:18
true then data will be the data that we got from the API
1:38:24
so we are getting the data from users table we are selecting the data then we are checking the ID and we getting the
1:38:31
single item so let's see if this is working so let's get the response from
1:38:36
this API get user data and we need to post the user ID which we can get on do
1:38:44
ID now let's console log this and see if we are getting the user
1:38:51
data and let's save this and uh we can see in the console we uh got the user
1:38:58
data at the bottom uh data has address bio created at email ID and the image we
1:39:07
have everything that we need so uh now what we can do is we can remove this console and uh first we will check if
1:39:14
the success is true on the response then uh we will uh set the user data into the
1:39:20
user state so now we can import our set user data which we created earlier I
1:39:25
know we can use a set o function for the same reason but I created this different function because we're setting the user
1:39:32
data we're not setting the O so let's use this function here to set the O state but just to be clear this doesn't
1:39:40
make any difference even if you use this function or the set o function this will work the same so now if you see in the
1:39:47
console we got the updated user and this has the address the bio image and the
1:39:53
name name of the user and this is coming from the users table this data does not include the session but we only need
1:40:00
these values to display in the user profile so now we're going to create all
User Profile
1:40:06
the tables and the policies in the super Bas so let's see uh what are the
1:40:13
steps okay we going to perform this step and first we're going to create an ER on
1:40:18
this platform to see how all the connect all the tables will be connected and all the property properties for each table
1:40:25
so let's create an ER which is an entity relationship
1:40:30
diagram and uh this is I believe a dummy diagram it has workspaces users folder
1:40:37
and in the sidebar you can see all the properties and I think we can change the
1:40:43
properties from here so let's say if I add bio in the users table this will be updated on the diagram so this is very
1:40:51
cool but we're going to use AI to create all the tables and the properties and uh
1:40:56
create relationships between them so I will just paste a description of our system so a social media system where
1:41:03
user can log in sign up create post like post comment on the post and then receive the notifications and we already
1:41:11
have the users table with all of these properties then I will create a post folder so each user can create one or
1:41:18
multiple posts and we'll have post likes then post comments so user can comment
1:41:24
on all of the post then finally we will have the notifications with the sender ID and the receiver ID and this will
1:41:31
create the relationships based on the foreign keys so let's see what do we get
1:41:37
so it's creating all the relationships and all the tables and the diagram is complete so I
1:41:44
think it's done a very good job at creating all of these relationships and all the
1:41:50
tables so users uh have all these fields which we defined then it's connected to
1:41:57
the post so each user can create many posts like this so then each user can
1:42:04
receive or send notifications then uh the comments are related to the users
1:42:10
and comments are also related to the posts uh with the post ID then we have
1:42:16
post likes which are also related to the post and then the users so it has done a
1:42:21
very uh good job at creating the relationship so if I need to change anything let uh let's say let's add a
1:42:30
city I can add that and this uh diagram will be updated so now we're going to
1:42:35
create all of these tables into our superbase project so uh let's create a
1:42:41
new table for the posts and we already have the ID and the
1:42:47
created add property so let's add the body property uh this will be the text
1:42:53
property and we are going to use a re text editor so this will store all the
1:42:58
HTML markup then we will have a file property uh which will also be a text
1:43:04
and then the user ID so this will be a uu ID which uh we have in the users
1:43:09
table so for that we will need to add a foreign key relationship so we're going to choose the public schema then the
1:43:16
table which we're going to reference to so in our case we're going to reference to the users table now we we need to
1:43:23
select the user ID from the post table and then we need to connect it to the ID
1:43:28
from the users table this way uh foreign key relationship will be created now uh
1:43:34
next uh we have these actions if the reference row is updated or removed so
1:43:40
if the user is removed we need to set the user ID as null but you can also use
1:43:46
Cascade option to remove the post if the user is removed so that's done now we
1:43:53
have all the properties and we have added the foreign key for the Post folder now uh let's save this and it's
1:44:01
going to create the table now superas has enabled the RO level security for this table and for that we will need to
1:44:07
create a policy so that user can read and write data into this table so let's
1:44:13
create a policy and we're going to create a custom policy so we can perform all of these operations uh let's name it
1:44:20
alow all and it will be only for the authenticated roles so only the
1:44:26
authenticated users will be able to perform all of these functions or whatever the function I want them to but
1:44:33
in the sidebar you can see we have the template so if I choose this uh this will enable all the users to read access
1:44:40
only but we don't want that we want to allow all the authenticated users to perform all of the steps so all of the
1:44:48
actions so I will choose this and I will save the policy either we can just
1:44:53
disable the RO level security or we will need to create the policies for all the tables like this now let's create the
1:45:01
post likes table and we already have the ID and the created add property now let's add two
1:45:08
more columns post ID and the user ID now these will be the foreign keys so let's
1:45:13
add the relation for them first with the posts so the post ID from this table
1:45:19
will point to the ID in the post table and if if the reference is removed
1:45:24
uh we will just set it to null and let's save
1:45:31
this now let's add another foreign key for the users table so the user ID in
1:45:37
this table will point to the ID in the users table and same if the reference is
1:45:44
removed we'll just set it to null and let's save this and uh let's
1:45:49
save the table okay so the table is created now let's
1:45:55
add an RLS policy so that users can add and read data from this table uh let's choose this template and
1:46:03
change the name to allow all and select all the options then
1:46:09
choose the roles as authenticated and save this
1:46:14
policy okay now let's create our comments table new table and name will
1:46:21
be comments ID created ad then we'll have a text property and uh this will be the type
1:46:29
text then we will have two foreign Keys as the user ID and the post
1:46:35
ID now let's add the foreign key relationships for both of them so uh we
1:46:41
need to point the post ID in this table to the ID in the post
1:46:46
table like this and same for the reference row is removed we'll set it to
1:46:51
null and save this now let's add uh the foreign key
1:46:57
for the user ID so the user ID from this table will Appo to the ID in the users
1:47:03
table and same for this section this will set it to null and let's save
1:47:10
this and let's save the table ensure that all the columns are
1:47:16
signed a type uh why why does the user ID does not have a type
1:47:23
CU we mistakenly edit this for the Post ID so we need to edit this reference and
1:47:28
this should be the user ID and let's save this so the relations are correct
1:47:35
now let's save the table and the table is created and let's
1:47:42
add a new table notifications we already have the ID
1:47:47
created at we're going to have a title property which will be the text and and
1:47:52
then we will have the sender ID who sent this notification and the receiver ID
1:47:58
who will receive this notification then we will have a data property to hold the
1:48:03
data this will be just a stringify Jon so let's add the relation so both of the
1:48:09
sender ID and receiver ID will point to the users table so the sender ID points
1:48:14
to the ID and the users and this action should be set all and save this let's
1:48:21
add for the receiver ID so this will connect to users receiver ID to the ID in the users
1:48:31
and if the reference is removed we set it to null let's save this and everything's good to go so let's save
1:48:38
the table and it's created now let's add the
1:48:46
r level security policy so let's create the policy choose the template all
1:48:53
all then choose all the options and for the roles select authenticated then save
1:49:00
policy okay so I think we do not have a policy for the commment table I think I
1:49:06
forgot to do that so if I go into the comments table uh we don't have a policy
1:49:12
so if I go into the notification you will see we have added a policy here so
1:49:17
let's add a policy for the comments table choose the template all operations and and this will be allow
1:49:24
all and then I use the roles as authenticated then save
1:49:32
policy okay so with this we have created all of the tables and their policies so
1:49:38
uh now we are ready to work on our home section so all of this is done so next
1:49:44
we're going to work on our home screen design so we have created all the
1:49:49
policies and the tables so let's create a mockup of our home
1:49:57
screen so this is our home screen and uh let's choose a blank
1:50:03
[Music] rectangle
1:50:08
um okay so this works so uh this will be our home screen and on the top we're
1:50:14
going to show the title of our app which is link up this will show here and then
1:50:20
we will have three icons on the right side so first icon will be the hard
1:50:25
[Music] icon here and then we will have a plus
1:50:31
icon uh this will work and uh for the third icon we will
1:50:38
have a user Avatar icon so let's use this one make it
1:50:44
smaller like this so all of these will be buttons so if we click on any this
1:50:50
will open a new screen so uh let's just align them correctly like
1:50:55
this so this hard icon will open the notification screen where users can see
1:51:00
all of the notification like who commented on their post then plus icon will open the new post screen and the
1:51:07
user icon will open the profile screen now let me just use these arrows to Appo
1:51:13
to these uh Straight Arrow and same for the new post
1:51:24
and the user profile like this just to make it clear
1:51:32
so this will be the header section of our home screen and we will Design the rest of the screen later so our first
1:51:39
step is to create the header section and before I start designing this I need to copy and paste all the styles for this
1:51:46
screen because if I type and explain all of these Styles this will uh make this
1:51:51
video very longer so I'll just copy this here and import all of these functions and the theme object and I'll show you
1:51:59
these Styles whenever we use them so that you can just copy them and use them in your application so here we will have
1:52:05
a container view so let's give this a style of container then inside this we will show
1:52:11
our header so let's create a view and give this a style of
1:52:19
header and inside this we're going to have a text comp component as the name of our application so let's give it a
1:52:25
style of styles. title so the name of our application is
1:52:31
link up so let's add it here and save this so you can see the title is already
1:52:36
styled because for the header we have these Styles uh we want to make it Flex
1:52:41
row so that we can show the icons on the other side and for the title we have these styles using our theme and HP
1:52:49
function so let's go back to header and here we just need to add another view
1:52:55
for the icons so styles. icons and inside this we need to add a
1:53:02
pressable and here first we need to add the icon so let's choose the icon
1:53:09
component and the name will be hard because we already have this icon
1:53:15
into our icons folder so for the size we're going to use HP of
1:53:21
3.2 and for the stroke width we'll use two and for the color let's use the text
1:53:28
color from our theme object and save this so we can see the hard icon and if
1:53:35
I show you the styles for the icons this will be in a flex row like this because
1:53:41
we are going to add other icons so these should show in a row so let's just copy
1:53:46
uh this pressable two more times for other icons and we'll change this to Plus
1:53:53
and just to user for now so there we have three icons on the right side so
1:54:01
whenever user presses on any of the button this will move to our new screen so let's create the screens for all of
1:54:07
them profile. GSX and we will have another screen as
1:54:16
notifications so let's create a functional component in all of them let's change the name to Capital
1:54:23
same for the profile and when user presses on these
1:54:30
actions we should move to the new screen and for that we will need to have a router so let's use the use router
1:54:38
hook and let's add an onpress method on this one and this will move to
1:54:49
Notifications now let's just copy this function for all of the buttons and this
1:54:55
will move to the new post screen which we haven't created we'll create it in a second and this will move to the profile
1:55:04
screen and let's save this now let's create this new post screen inside our main
1:55:10
folder new post. GSX and let's create a functional
1:55:15
component and change it to capital name like this so for this user I icon we're
1:55:22
going to show the user image and for that we're going to have another component as the Avatar this will show
1:55:29
the user image and if he doesn't have any this will just show a default image
1:55:35
so let's just save this and import this
1:55:42
here like this and now this shows the Avatar so let's go to this and start
1:55:47
designing this will receive some properties as the URI which will be the image URI and then the size and the
1:55:55
default size will be HP 4.5 then we will have a rounded
1:56:01
property to make the image rounded and this will have a default value of radius. MD from our theme and a style
1:56:10
object if we want to have any so let's remove this and here we're going to use
1:56:15
the Expo image so let's import it I think we haven't installed it yet so
1:56:21
we're going to use Expo image because this has the image cache so we can use that
1:56:27
so this is the Expo image from Expo Dev website so let's just copy this command
1:56:33
and install this quickly so with image caching it will
1:56:39
load the image once and after that it will show the cast image and that is very helpful for the performance so we
1:56:47
will use this image now and this has all of these properties which are mostly
1:56:52
similar to react native image component so let's use this and let's import the Expo image
1:57:01
here which is uh not Auto importing so let's just import it
1:57:07
manually image from Expo
1:57:12
image like this now let's add the properties and we will have a source we
1:57:18
which will be the URI and this will be in double bracket like this and then we will have a
1:57:25
transition property which will be 100 millisecond so this is just a transition
1:57:30
uh to until the image shows and then we will have the styles. Avatar which we
1:57:35
will Define in a second and then using the props that we are passing we need to
1:57:41
specify the height and the width of this image mod radius will be rounded and
1:57:48
then uh any extra style that we want to pass so we can just override it into the
1:57:53
parent component let's close this and now Define this Avatar Style here so
1:57:59
first we'll have the Border curve as continuous then the Border color will be
1:58:07
the dark light color from our theme and then the Border width will be
1:58:13
one like this and let's save this so we can see the Border width and the image
1:58:19
is blank because we are not posing the URI from the parent component so let's go into the parent component and here
1:58:27
we're going to use the user image so if I open the console you will see we have this user object and here we must have
1:58:34
the image property which is uh for now it's null but later we will upload the
1:58:39
user data and this will show the image so we will pause this property URI as
1:58:45
the user. image and then we're going to pass the size so we can control the size from the
1:58:51
parent components and let's use HP of 4.3 like this and the rounded property
1:58:58
to make the image rounded let's use theme. radius do
1:59:03
small and we can pause any extra style if we want to overwrite the default Styles so let's use the Border width of
1:59:10
two now we can see our Styles so we're processing the user. image but it is not
1:59:15
showing here so we need to have a way to show the default image so let's create a new service uh let's call it image
1:59:25
service and let's create a function here that will return the user image source
1:59:30
so let's call it get user image source and this will receive an image
1:59:38
path and we will make a condition here so if we have the image path in that case we will return an object so uh this
1:59:46
object will have a URI as the image path like this and and if we don't have the
1:59:52
image PA then we will turn the default image from our assets so require the
1:59:58
image from assets SL images SL deault user.png
2:00:06
like this now let's import this function and
2:00:11
use this inside this image component instead of using the Ur So get user
2:00:18
image source and we need to pause the URI and save this so with this we can
2:00:24
see the default user image and that's because right now we don't have the image property on our user data and
2:00:32
because we don't have that that's why we are seeing this default image but once we update this we will see the user
2:00:38
image from the database so let's give this container a background of white
2:00:44
from this screen wrapper like this and our uh new screens should be
2:00:50
working so we can go to new uh notifications or we can go to the new post so the header section of our home
2:00:57
screen is complete and at the bottom we're going to show the user post but we will do that when we have the post
2:01:04
inside our database so let's just skip all of these steps now and move to our profile screen so uh let's create a
2:01:12
mockup screen of our uh profile so this will be our profile
2:01:19
screen let's add a label here here user profile like this and first we will have
2:01:27
a back button at the top left corner so let's use an
2:01:33
arrow like this and we will have a back button then we will have a user Avatar
2:01:40
at the center on the top here this will be big user image then after this we're
2:01:47
going to show the usern name at the bottom of this image this will display here
2:01:52
and after this we're going to show the location this will be a very small text this will show after the username we
2:01:59
also going to have a log out button on the top right corner so let's use the circle icon for that and this will show
2:02:09
here this will be our log out button so let's add a label for
2:02:16
this let's make it big and let's point to this using an arrow
2:02:22
like this so after this username we're going to show some more user info as
2:02:28
user email then we have the user phone number and the user bio so this will
2:02:34
show after the username and the location on this left side so this will be just
2:02:39
the user info and after this we're going to show the user Prof uh user posts in
2:02:44
this section but we will do that later so for now this is the user info and uh from this section user will be able to
2:02:51
edit Ed his information so let's add an edit icon on the user image so first
2:02:59
we're going to start creating this header section for our user so let's go into the profile
2:03:05
section and here first we will use our screen wrapper and let's save this now we're
2:03:13
going to use the user data on this screen so first let's uh get the user
2:03:18
and set o function from our use Au hook and we will also need the router on this
2:03:25
screen so let's create a router using a use router from exper
2:03:31
router now we're going to create the user header into a separate component and then use this into the main
2:03:38
component I'll explain later why we are doing this but for now just do the same
2:03:43
so this will receive the user and a router and this will return a
2:03:48
view let's give this a style of Flex one and a background of
2:03:55
white then inside this let's use a text component that will say user header for
2:04:01
now and uh let's use this inside our screen wrapper and pause the user and the
2:04:09
router properties like this and let's save this
2:04:17
and we also need to have a background of white for our screen wrapper
2:04:23
okay so now we're going to create a common header component that we can use on multiple screens so let's define it
2:04:29
into our components let's create a functional component and save this now
2:04:34
let's import this in here let's just create a view and inside this import our
2:04:40
header component that we just created this component will show a title
2:04:45
and a back button so for this screen it will be profile so let's go into the
2:04:50
header component and first we will show a back button here and then the title for this header
2:04:56
section so let's receive the title and then we will have a property show back
2:05:01
button by default it will be false and then we will have a margin bottom
2:05:07
property and by default it will be 10 pixels now let's create a router so that we can go back from uh this back
2:05:15
button so let's add a styles for this container as styles. container
2:05:22
and then we are also passing the margin bottom so let's add margin bottom as MB
2:05:28
like this so uh let's H create a back
2:05:33
button using this property so if show back button is true only then we're going to show the back button so let's
2:05:40
create a view give this a style of styles. back button and inside this we
2:05:47
will have our back button which we already have created so let's import our
2:05:52
back button and pause the router like this and let's close it and save this so
2:06:00
if I save this uh we won't see anything because by default the showback button is false so for the title let's add a
2:06:08
text let's give it a style as styles. text and inside this we will show the
2:06:15
title uh if we don't have then I will just show an empty string like this so
2:06:21
so let's add the styles for the container let's use flex direction as
2:06:27
row justify content Center align items into the center as
2:06:35
well then we will have a margin top of 5 pixels and a gap of 10 pixels so let's
2:06:43
save this so now we can see the profile text so let's add a styles for the profile uh
2:06:51
title so title will have a responsive font size using HP of
2:06:58
2.7 then let's add a font weight let's make this uh this semi board using our
2:07:04
theme object so we have all of these uh predefined fonts into our theme object
2:07:10
so if I go into this theme we have medium semibold extra Bard so let's use
2:07:17
the color as the text dog color from our theme okay so the title looks good and now we
2:07:25
can go into the parent component and show change this show back button to
2:07:30
true so let's add this property here show back button and make it
2:07:37
true so we can see the back button but uh it shouldn't display here
2:07:44
so we need to have the back button Styles so let's change these styles to back button
2:07:52
and uh let's make it absolute so back button will have the position as
2:07:59
absolute and the left property as zero so that it shows on the left side like
2:08:05
this so now we're going to have a pedding around the content into the main
2:08:10
screen so let's go into the profile section and inside this container let's
2:08:15
use a padding horizontal of 4% using our WP function
2:08:22
like this so now we have a pedding so if I click on the back button we should go
2:08:28
back to the previous screen and it's working now let's add a log out button
2:08:33
after this header so let's add a touchable opacity we're going to make it absolute
2:08:39
so that it shows on the top right corner so let's give it a style of log out [Music]
2:08:47
button then we're going to have a function handle log out
2:08:52
so inside this button we're going to show the log out icon so let's import our icon component and name will be log
2:09:00
out and the color will be the rose color from our theme
2:09:05
object like this now let's add this handle log function in our profile
2:09:12
component this will be an async function and we're going to later add a confirm
2:09:17
model to confirm if user wants to log out so let's just add a M for now and
2:09:23
let's save this okay so handle log up doesn't exist
2:09:28
uh did we got the name right okay so we need to pass it into
2:09:34
the user header so that we can access it from this component right
2:09:41
here so now we can see the log out button but we need to have some styles
2:09:46
for it so I'm just going to copy and paste the styles for this whole component
2:09:51
and I'll show you the Styles whenever we use them so you can just copy and use in your application so let's just import
2:09:58
this WP and HP function and the theme is already
2:10:04
imported so let's save this and now you can see the button is styled and it's on the top right corner because we have the
2:10:11
following styles for this the position is absolute so now when we click on this
2:10:17
icon it should confirm the user that if he really wants to log out so we're going to use alert from react
2:10:25
native and the title will be confirm and we're going to display a
2:10:32
message are you sure you want to log out and for the third option uh we can
2:10:40
have an array of the options that we want to display so uh each option has
2:10:46
some properties so the text that we want to show then we have an onpress function
2:10:51
which executes when we click on this button so for now it will just console L
2:10:57
message model cancelled then we have a style property which uh we have predefined values for so we can use
2:11:04
cancel style for this button then uh we will have a log out button this will say log
2:11:11
out and on press method will trigger an on logout function which will'll create
2:11:17
in a second and for the style we will use destructive so it will give like a red
2:11:24
color to our button so for the log out function we already have that into our home section
2:11:30
so let's just copy this own log out function and paste this into our profile
2:11:36
screen now let's import this webas client and everything else seems all
2:11:44
right so let's save this and see if this is working so now if I click on this
2:11:50
icon we we see the model and if I cancel it we see model cancelled now let's see
2:11:55
if the log out functionality is working and we're moving to welcome screen the user is null so everything is
2:12:02
working perfectly so let's go to login and login with my
2:12:15
account okay so now we don't need this log out on the home screen so let's just
2:12:20
comment hit down we need to comment this function and we also need to comment this button
2:12:27
at the bottom okay so now let's move to home screen and start showing the profile
2:12:35
data on this screen and in here we're going to create
2:12:41
a new container so let's create a view with a style of
2:12:49
container and inside this let create another view and give it a gap of 15
2:12:55
pixels then uh here we're going to show the profile image so uh let's create uh
2:13:02
let's just import our Avatar component that we created earlier and let's give
2:13:07
this a URI of user. image and we already receiv receiving
2:13:14
this user from the parent component then the size will be HP of 12 this is going
2:13:20
to be a big image of the user now we also going to make this rounded so we're
2:13:25
going to need a bigger border radius so what we will do is use double XEL from
2:13:31
our theme and we will multiply it by 1.4 and let's close it and save it so
2:13:37
now we can see the default user image but it's not into the center so let's create another view and give it a style
2:13:45
of Avatar container which we already have into our stylesheet and let's move
2:13:51
this inside this container okay so let's save this and
2:13:58
now the image is into the center and we have the following styles for the Avatar
2:14:03
container so align self to the center so that it show it shows into the center
2:14:10
now we're going to create another button for the edit profile action so let's
2:14:15
create a pressable then import the icon name will be edit and let's use a stroke
2:14:22
width of 2.5 and then the size will be 20
2:14:29
pixels let's save this so there we can see the edit icon but let's add the
2:14:36
styles for this style will be styles. edit icon and let's save this now it's
2:14:41
showing on top of the image because it has a position of absolute and have the following Styles which you can copy and
2:14:49
we also have a shadow on this icon because uh the background is wide and uh
2:14:54
the background of the screen is wide so that's why I added a shadow uh let's
2:15:00
move to the edit profile screen when we press on this icon so router. push to
2:15:06
edit profile which we will create later so let's save
2:15:11
this so we need to have some spacing between the profile and the user image
2:15:16
and uh this profile is coming from the header so let's go into the header
2:15:22
and uh this needs to be true by default so let's make it true and we can pause margin bottom to have a little spacing
2:15:29
so now we don't need this property it's true by default and let's add a margin bottom of
2:15:36
30 and let's save this now we have a spacing between these two now let's add
2:15:41
another view for the username and the address of the user so uh let's just
2:15:46
make a comment username and the address
2:15:53
then uh let's create a view and give this a style of align
2:15:58
items into the center and let's add a gap of four
2:16:03
pixels now let's add a text and give it a style of
2:16:12
username and then here we already have the user data so we'll just display the
2:16:17
usern name so if we have the user then display the the username and let's save this so we can see the username from my
2:16:25
account now for the username we have the following Styles now let's just copy this text one
2:16:32
more time for the user address and let's change this to infotext which is a style
2:16:38
for small text and change this to address so for now we don't see the
2:16:43
address because we don't have this into the user data so if I remove this and
2:16:49
add New York here you will see this is how it will show the address of the user
2:16:56
so now let's add another view after this to show the user email phone and the
2:17:02
bio let's add a view and give this a gap of 10
2:17:08
pixels and we're going to display the user email here so let's wrap it inside
2:17:14
another view and give it a style of info so we're going to add an icon and then
2:17:19
the email so we have the following style for the info so let's add the
2:17:26
icon let's use the icon component and give this a name of mail then the size
2:17:33
will be 20 and the color will be the text light color from our theme
2:17:39
object and let's save this so now we can see the email icon now let's add the
2:17:46
user email so text and give this a style of info
2:17:57
textt then here if we have the user then we'll display the user email. email like
2:18:03
this and this will um it should display the username but it's not for some
2:18:11
reason so let's see in the console we have the user data address bio and the
2:18:18
email is null okay okay so let's see in
2:18:23
the super base we should have that into the users table and we don't have the email here
2:18:30
so let's see uh into our function because our function is basically storing this data so let's edit function
2:18:40
so here is the issue we stored the user ID and the name but we forgot to add the user email so we can handle this
2:18:47
situation in two ways one is to delete all the users then modify this and then
2:18:52
register all the users again or we can find a patch on our front end
2:18:59
so one way to handle this is to go into our layout and here we are getting the
2:19:05
user session so I think we can use the email from the session data so let's
2:19:11
console log the Au user and let's see what data do we
2:19:18
getting into the session user okay so let's see in the console we
2:19:26
have um we have the ODS user here and we do have the email property on this user
2:19:32
so if I console log the email and save this so here we have the Au user and
2:19:39
this is my email so now we can use this uh user data when we update the user
2:19:46
data into our context so let's go into this function uh sorry this set user
2:19:52
data function and here uh we can just add the
2:19:58
email as the user. email because when we are setting this we already have the
2:20:04
user from the session so this will just save the user email so let's just
2:20:09
refresh the app and the email is still
2:20:16
undefined um maybe the user session data was still null at this point so let me
2:20:22
just uh remove all of the console logs because the terminal is a bit messy so
2:20:29
let's go to home and comment this user as well now let's try to console log the
2:20:35
session user when we are updating this user data see if we actually have any user
2:20:42
data in this state so uh this will be the session
2:20:48
user data so let's save this and all user is null so we can't do this uh we
2:20:56
will need to find another way we don't have the user session data here but we
2:21:01
do have this inside this use effect so we can just pass the user email to this
2:21:07
function and then receive the email here so we can get the email on session.
2:21:15
user. email and we can pass this uh email alongside with the response data and
2:21:23
this will update the user State into our context so let's save this and in our o
2:21:31
context now we don't need to set this email here so let's just remove this and
2:21:36
save this and we need to uh uncomment this user state to see if you're are
2:21:43
getting the email and yes we do now we have this email property and this will
2:21:49
now show on the profile file so this is not the best way to implement this we're only doing this because I made a mistake
2:21:57
while I was creating the trigger and the function so when you do this you make sure you add the email as well so you
2:22:04
don't have to do this so now let's just copy this for the user phone
2:22:11
number and for the icon we're going to use the call icon and here we're going to display the
2:22:18
user phone number but for now we don't have his phone number on this data so it
2:22:24
won't display so let's make a condition here if we have the user and the user
2:22:30
phone number only then we will display this block so that if if we don't have
2:22:36
the user phone number we don't even see uh this call
2:22:41
icon so let's save this and we don't see the call icon now at the bottom we are
2:22:47
going to display the user bio so let's make a condition here as well if we have
2:22:53
the user and the user bio then we're going to display a text and this will
2:22:59
have a style of infotext and this will just simply
2:23:05
display user. bio and let's save
2:23:10
this so we won't see the bio for now because we don't have it on the user data but if I remove this condition and
2:23:19
uh in here I just add a Rand text this is how the bio will show but we will implement this when we implement the
2:23:25
edit profile screen so if I I click on this edit button this will move to the
2:23:31
edit profile screen which we haven't created yet so uh let's go into our main
2:23:36
folder and create edit profile then let's create a functional
2:23:44
component let's change this to capital name and let's save this so let's also
2:23:51
import the screen wrapper so that we can see the text properly let save this and
2:23:56
now if I I click on the edit button we move to the edit profile screen so uh
2:24:04
before we start developing let's create a mockup of the edit profile screen in
2:24:09
our editor here so the header is done log out button is done user details are
2:24:15
done uh we work on the user post section later but for now uh we need to work on
2:24:21
the edit profile screen so uh let's create an edit profile screen
2:24:28
here let's move this to the side of the profile in here so this screen will open
2:24:36
when we click on the edit icon so let's point to this like this and this will be
2:24:43
our edit profile screen and on the top left corner we
2:24:49
going to have a back Arrow icon to go back to the previous screen so let's add
2:24:54
this icon here then we going to also have the user profile image here uh not
2:25:02
this one uh let's use this one so this is also going to be a big image of the user
2:25:09
and uh if user clicks on this we'll we will be able to pick any image from the
2:25:15
gallery so after this we're going to have a text box to edit the name of the
2:25:20
user this will say name now let's copy this for the user phone
2:25:27
number and for the user address as
2:25:33
well then we're going to have a multi-line input for the user bio so it
2:25:38
will show like this and user will be able to enter his bio at the bottom
2:25:44
we're going to have this button that will update the user data this will say update and we going to match it to our
2:25:51
theme like this so when we click on this this is going to update the data into
2:25:57
the users table and also we will update the user State into our context so that
2:26:02
we can use this data anywhere into our application so now we're going to design
Update Profile
2:26:08
our edit profile screen and before we do that let me just copy and paste the styles for this
2:26:14
screen because if I just try to type every style this will make this video
2:26:19
very longer so let me paste these Styles
2:26:25
here okay so these are the Styles we're going to use for this screen mostly these are similar to the user profile
2:26:31
section so let's create a view and give this a style of
2:26:38
container and inside this we're going to have a scroll view so let's give this a
2:26:43
style of Flex one and here here we can just import our
2:26:51
header component for our edit profile screen and we're going to give this a title of edit
2:27:00
profile like this and if I close this and save this okay so we have an error so we need
2:27:08
to import this WP this HP function and I think we also need to
2:27:14
import our theme object yes so let's save this and now we can
2:27:19
see we have a nice header on the profile screen so for the container we have the following Styles and uh now let's add a
2:27:28
form so let's create a view and give this a style of
2:27:35
form then let's create an avatar container for the user image so let's
2:27:41
give this a style of Avatar container and inside this let's use the
2:27:47
Expo image so here is export image just import it and give this a source of the
2:27:55
user image so we're going to uh add the image source later but for now let's
2:28:00
give this a style of Avatar so for the image source we need
2:28:06
to get the user data from our o context like this and let's create the
2:28:14
image source for now we're going to use the get user image source function and we're
2:28:19
going to pause the user. image but later we're going to use the local image so uh we'll replace this inside the image
2:28:26
source variable now let's give this H container a background of white and now
2:28:33
we're going to create a camera icon on this image so uh this will let user know that he can press on this icon and
2:28:40
choose the image from the gallery so let's create our icon and name this as
2:28:47
camera then the size will be 20 and let's give this a stroke width of
2:28:55
2.5 and let's save this so there we can see the camera icon
2:29:00
but let's give it it a style of camera icon from our stylesheet and now it's
2:29:06
placed accordingly so this has a position of absolute and has some Shadow properties same as the edit icon uh in
2:29:13
the user profile section so now when we click on this button it will open up
2:29:20
image pick model so let's just create this function here and we will implement
2:29:25
this later but for now let's just leave it empty like this and let's move to the next so now
2:29:34
uh let's create a form and first we're going to have a a text so let's give
2:29:40
this a font size of HP 1.5 and the color will be the text color
2:29:46
from our theme and this will say please fill your
2:29:52
profile details then we going to have a name
2:29:59
input so that user can update his name so let's import our input component from
2:30:04
the components folder and let's give this an icon and we can use our icon component
2:30:12
uh let's use the username sorry user icon and let's close it uh then we are
2:30:19
going to have a placeholder property this will say enter your
2:30:26
name let's leave the value empty for now and uh let's use an empty function on on
2:30:32
change text function so let's just close it and save
2:30:38
it um let's use null for now okay so we can see the input now
2:30:44
let's create a state for all the user values so uh let's create user and set
2:30:50
user and this will have a value of an object with the following properties
2:30:56
name and the phone number image as null and bio and the
2:31:05
address so now we have two users so let's change this to current
2:31:12
user and uh we also need to change this user. image to current user like this
2:31:21
um actually we don't need to do this so let's just change this to user. image and we're going to set the user values
2:31:28
uh in the use effect hop so whenever we open this screen and we have the values
2:31:34
for the current user we will check this and we will also add this to the dependency array and whenever we have
2:31:40
the current user values we will set the user values based on the current user so name will be a current user.name or an
2:31:50
empty string so same for the phone number current user. phone
2:31:55
number or an mty string image will be uh current user.
2:32:03
image or null and same for the address and the
2:32:09
bio
2:32:20
so now this will set the user with the current user values and uh this will
2:32:25
show the user image so let's save this and now we can use this user name into
2:32:30
the name input so value will be user.name and here we will set the
2:32:37
user with all the rest of the user properties the name will be this
2:32:44
value and if I save this you will see the current usern name so let's just copy this input for for the phone number
2:32:51
address and the bio so let's change this icon to call and this placeholder will say enter
2:32:59
your phone number okay let's save this and now we can see all the inputs so this will be
2:33:06
user. phone number and this will also set the phone number
2:33:12
here and let's save this So currently we don't have the phone number so currently
2:33:17
it's showing empty so this icon will show the location icon and this will say enter your
2:33:24
address then we also need to change this address in both places then for the last
2:33:31
input we are going to show the user bio so we don't need the icon for this so let's just remove this property and this
2:33:37
will say enter your bio this will be user. bio and then we're going to set the bio here so let's save this now
2:33:45
we're going to make it multi-line so this will show as a multi-line input so for that we will set the multi-line
2:33:51
property true then we're going to add the container style property so this
2:33:57
will basically override the container styles for the input so here you can see we have the container then container
2:34:03
Styles these are the default Styles so uh this will be container style not
2:34:10
Styles so now we can overwrite this and use styles. bio from our
2:34:15
stylesheet and this will have a multi-line style so now if if I show you these Styles we have Flex Direction row
2:34:22
height align items and a bading vertical so you can just copy this so now after
2:34:29
this we are going to have a button at the bottom which will update this uh these properties so let's import our
2:34:35
button component and the title will say
2:34:41
update then we're going to pass a loading property which we'll create in a second and then when we click on this
2:34:49
button this will trigger an onsubmit function so let's close this now let's create this uh loading
2:34:56
State and set loading by default this is going to be
2:35:01
false and now let's create uh this on submit function this is going to be an async
2:35:09
function and inside this first we're going to clone the user data from this user
2:35:16
State like this and now let's get the name phone number address and the bio
2:35:24
from this data so first we're going to make a
2:35:31
check if uh we have all of these values uh if not then we're going to prompt the
2:35:36
user please fill all the fields so let's check if we don't have either of these
2:35:43
values now let's not include the image we will handle the image later uh we're going to use alert to alert the user
2:35:50
about a profile and we will display a message please fill all the fields like this and
2:35:59
we will simply return from here but if we have all of this data then we going
2:36:04
to Simply start the loading and then implement the user update so for now
2:36:10
let's just add a comment update user let save this now if I click on this update
2:36:16
button we will see this message because we don't have the dat in all the fields
2:36:21
so we'll still see this m message so let's fill all the fields now if I click on this we see the loading so now it's
2:36:29
time to implement the update user function so let's just refresh go to profile edit
2:36:36
profile so let's go into our user service uh let's just copy this
2:36:44
function I change this name to update user and this will save the US user ID
2:36:50
and the data that we want to update so we're going to call the update function
2:36:55
which will return the error only so if I go to the super base uh let me show you
2:37:02
how this works so if I go to the update data here you can see this function returns an error but you can uh call
2:37:08
this select function to get the data as well but uh we already have the data so
2:37:14
we'll just simply call this update uh function so uh let's remove this and
2:37:21
here just call update function and pause the data that we want to update then we
2:37:27
going to use the EQ function so this will update the data where where this where the ID equals this user ID and if
2:37:35
we have any error this will return the error otherwise this will return the data so let's save this and now let's uh
2:37:43
try to use this function let's go into the edit profile and uh let's go to edit profile screen
2:37:50
so here we will call this function and this will give us a response so let's
2:37:55
import the update user function and we're going to pass the current user ID and then the user data
2:38:03
that we want to update and for now let's just console log the result to see if you are getting
2:38:10
the success response we also need to stop the
2:38:16
loading when the API call is finished so set load recing to false let's save this and see if this is
2:38:24
working so open the console and for now if I click on this we see this error but
2:38:30
let me just put the dummy data into the fields I live in New York I'm just
2:38:35
kidding I'm from Pakistan so he codes and then update so here in the console we are
2:38:43
getting Su success as true so let's see if it's actually updating into the super
2:38:50
base so let's go into the users table okay so the data is updated we can
2:38:57
see the bio the address and the phone number but the email is still null and that's because we made a mistake while
2:39:04
implementing the trigger and the function but it's already fixed on the front end so it's no big issue so now we
2:39:11
need to update the user Au State data as well so if I go into the Au context we
2:39:16
are going to update this user State using this I set user data function so
2:39:22
let's go into the edit profile let's get this
2:39:27
function here uh set user
2:39:32
data and when we update uh this user data into the super base we also need to
2:39:38
call this function so this will just update the user data for our local usage
2:39:43
so let's pause the current user values plus we will also spread the user
2:39:49
dat that we just updated and after the update we need to
2:39:54
go back to the previous screen so router. back uh so let's add this
2:40:01
router using our used router hook like this and let's save this so
2:40:09
this will update the data and move us back to the previous screen so let me just add the dummy data for my profile
2:40:16
and see if this is working and hit
2:40:22
update so we move to the profile screen and we can already see the data is updated for the phone bio and the
2:40:29
address and even if I refresh the app we'll see this updated data in the profile section now if we open the edit
2:40:37
uh profile screen this will prefill all the data that we previously updated now we're going to work on the user image
2:40:44
update So currently if we click on this camera icon this triggers this on pick
2:40:50
image function which is empty but we're going to open the image gallery using uh this Library Expo image picker so uh
2:40:59
this is the library and this opens the images or the videos into the gallery so
2:41:04
first let's install this Library using this command open the terminal and paste it
2:41:17
here okay so it's installed now let's close this and minimize this now we're
2:41:23
going to import this Library first so let's go into the library documentation here let's see for the code snippet okay
2:41:30
here so let's just copy this and import this into our edit profile
2:41:36
screen and first we are going to open the image gallery so here is the code
2:41:42
snipper for opening the image Library so let's use this so let's
2:41:47
create a result variable then let's use this image
2:41:54
picker do launch image Library um this is the function and here
2:42:02
we're going to pause some options so let's just copy these options here these are the options and let's just paste
2:42:09
them here so basically we are telling the image picker to pick all the images
2:42:14
with following settings but we only want to pick the images and instead of all we
2:42:19
can choose the images now it will only show the images from the gallery then for the quality let's choose 0.7 so this
2:42:27
will reduce the quality a little bit for the picked images so let's save this and
2:42:32
now if we click on this camera icon uh this is opening the image gallery we can see all the images so
2:42:39
this is working now once we get the result from this image Pier uh we are going to set the image into the user
2:42:46
state so let's just copy uh the this code so if the result is not cancelled
2:42:53
then we will set the uh image into the user so let's use set user and let's
2:42:59
just copy this result so first we will need to spread the user data um what happened here
2:43:06
let's go to edit profile here then for the image we're going to uh store the
2:43:11
image as the object not the URI so let's remove this URI so because this is the
2:43:17
local image so now here we can make a condition if the user. image uh if we
2:43:23
have the user. image and the type of the user. image is an object that means uh
2:43:29
the uh user image is picked from the local storage so in that case we going
2:43:34
to return the user. image. URI otherwise we can just get the user image source
2:43:41
from the super base so now if we pick any image from the gallery this image source will show that instead of the
2:43:47
default user so uh let's choose this image and it's working this is showing
2:43:54
the local image from the gallery but if we don't have the local image then get user image source function will show the
2:44:01
default image or the image from the super base so this is how this image picker is working now let's start
2:44:09
working on the image upload and first we will need to include the image field into this if statement to make it
2:44:16
required now let's make a condition here and check if the type of the image is uh
2:44:23
an object that means a user picked uh an image from the local Gallery so we need to upload this image and let's see what
2:44:31
we have done so far so we have uh implemented the header the user data
2:44:36
fails now we're still implementing the update so now to store the images we're going to create a bucket inside the
2:44:43
superus storage so let's go into the storage and then create a new bucket
2:44:50
let's name it uploads but you can name it up whatever you want so let's make this public and hit
2:44:58
save so here we can just create uh folders and then files inside them but
2:45:03
before that we will need to create some policies to create and read data from this bucket so let's create the new
2:45:09
policy uh you can uh get start quickly or you can choose the custom way to
2:45:15
create the policies so just click all the operations select insert update or
2:45:21
delete then uh let's name it allo all all users then for the Target roles just
2:45:28
select authenticated so all the authenticated users will be able to perform all of these operations and
2:45:36
everything looks all right so let's save the policy so now we'll be able to upload any data
2:45:43
into this so let's go into our application and first we will need to
2:45:48
create an upload functions so let's go into the image service and uh here uh sorry this is
2:45:55
user service image service and let's export a function upload file and this
2:46:02
is will this will be an Asing function which will receive three parameters so folder name where we want to store the
2:46:09
image and then the file URI for the file data and if the file is image or not so
2:46:16
by default this will be true so later we can also upload the video files using
2:46:21
this so let's create a Troy cach blog and if we have any error we just console
2:46:27
log the error as file upload
2:46:33
error then we going to return of success false response uh with a message that
2:46:40
will say could not upload
2:46:45
media like this so now we need to get the file name and that will also include
2:46:50
the folder name so basically inside the superb storage in the bucket we can
2:46:56
create folders uh so let's create images and when we save files inside this uh we
2:47:02
will need to pass this key as images SL the file name then we'll be able to get
2:47:08
this image but the folders can be created at runtime so we don't need to create them now so now let's create a
2:47:15
file name and we will get this using a function and let's call it get file path
2:47:22
and we're going to use the folder name and if the file is image or not because
2:47:27
the file name will also include the extension so we'll need to check if uh we want to have the image extension or
2:47:34
the video so we will receive the folder name and is image
2:47:39
property and here we'll just return a string with a folder name and the file
2:47:45
name so back TI and then slash first we'll use the folder name then after the
2:47:51
slash we are going to return the file name but this will be a random string plus the file extension so uh this whole
2:47:59
file path will be SL folder name slash file name so for this random string
2:48:05
we're going to use the get time function on this date object uh this will return us the
2:48:11
seconds and this will be unique every time for every file then after this we're going to add the file extension
2:48:18
using is image property so if is image is true that means we are uploading a
2:48:24
PNG file or the image file but if it's not then we are uploading the mp4 file
2:48:30
so if we uploading the file into the profiles folder then it will return profiles slash uh the second string plus
2:48:38
PNG extension like this and if you're uploading any file into the images
2:48:44
folder then it will return Images slash seconds and these seconds are basically uh
2:48:51
coming from this get time function then PNG extension so this was just for
2:48:57
demonstration I hope this makes sense so this will return us the file uh
2:49:04
path so now let's see how we can use this file name and the file Ur to upload
2:49:10
the image into super base so let's go into the superb storage service let's
2:49:16
see the documentation storage and we need to upload a file so let's click on
2:49:24
this so here if I zoom in you can see uh we using the Avatar bucket then public
2:49:31
SL Avatar is basically the same thing that I explained earlier public is the folder and Avatar 1.png is the file name
2:49:39
so this is how this will work but there is one more thing so if you see on the
2:49:45
side for react native using the blob data file or the form data does not work
2:49:51
so uh we'll have to upload the file using array buffer from B 64 file data
2:49:57
currently we only have a URI for the local file so we will have to convert it to base 64 then we will need to decode
2:50:05
it into an array buffer so uh first uh let's see how we can convert the file
2:50:12
into base 64 so let's open the console and we're going to install another library and library is called Expo file
2:50:19
system uh here is Library so let's just copy this command and install this
2:50:25
quickly so if you see the usage or the documentation you will find how you can
2:50:31
convert or use this uh Library okay so first let me just uh
2:50:39
copy the import statement here so this is how you import the file system then
2:50:44
uh when we get the file name after this we are going to get the file base 64
2:50:50
data so let's create a variable and use the file system and we
2:50:56
can use a function to read uh the files as string so read as string async then
2:51:02
we're going to pass the file URI and we're going to pause some options so uh
2:51:07
we need to specify the encoding in which we want to read this file so file system do encoding
2:51:14
type do base 64 so this will read uh this file as base 64 data so now this
2:51:21
way we will have the base 64 data now we need to convert this into the image data
2:51:27
uh sorry the array buffer so we'll need to decode this to array buffer and for
2:51:33
this we will need to install that library that super base suggested so let's open the terminal and go to super
2:51:42
base and library is B 64 array buffer let's copy this and let's just quickly
2:51:49
install this okay so it's installed now let's
2:51:55
use the decode function from this Library this one and let's just minimize
2:52:01
this terminal and let's close the second one so we're going to pass the file base
2:52:07
64 data into this function and this is going to convert it and give us an array
2:52:12
buffer for this file and now we can use this data to upload the file to super
2:52:18
base so we're going to use the superbase storage functionality to upload the file
2:52:23
like this and this will give us a response and we're going to get a data and an error property on this
2:52:30
response so let's use superbas dot uh let's move to the next
2:52:35
line do storage um this one and then we're going
2:52:41
to call the from function so here we can pass the bucket name we already have a bucket called uploads but if you have
2:52:48
have created with a different name you will need to pass that here then upload function here we need to pass the file
2:52:55
name which we have here so this includes the F folder name and the file name with an extension so then we need to add the
2:53:03
image data so first the file name then file data and then some options so now
2:53:09
we're going to specify this content type function which is very important because I I didn't use this before and I was not
2:53:16
able to show the image or the video So based on the file type we need to specify this and for the images it will
2:53:23
be images SL static for all the images and for videos it will be video/ startic
2:53:29
let's also add these options as gach control and upsert as false so if I add
2:53:36
upsert as false this will make sure it will create a new file every time we upload a file so that's fine so let's
2:53:43
save this and now uh we will check if we have any errors then we will return add
2:53:49
error response so let's return actually let's just return this response cuz it's
2:53:55
the same error response and if we don't get any error that means the file is uploaded
2:54:02
successfully and in that case we're going to return a success response and we have a Path property on the data that
2:54:09
we got from super base so we're going to return that in data property and we will
2:54:15
store this into database so that we can get the file using this path and this
2:54:20
path basically contains the folder name and the file name so let's console log
2:54:25
this data when we get it from the super base and we can see this dummy response here so here um we have response
2:54:35
somewhere uh here so here you can see data contains a path and this contains
2:54:40
the folder and the file name so this function is complete and hopefully this will work as expected so let's move move
2:54:48
to the edit profile and here let's use this function so this will give us an
2:54:54
image response and let's use this function upload file we're going to
2:54:59
store all the profile images into the profiles folder so folder name will be profiles file URI will be image. URI and
2:55:09
this image will be true because we're uploading an image then we will check if
2:55:14
the image response. success is true then we need to set the user data. image
2:55:20
property uh that we have here to the path that we got from the super base so
2:55:25
image response. data like this and if the success is false that means uh the
2:55:32
image was not uploaded and in that case we will just uh set the user data do
2:55:37
image to null you can also throw an error from here but uh I will do this so let's save
2:55:46
this and see if this is working so let's choose an image um let's choose this
2:55:52
one and then hit update there's the loading and we move
2:55:58
to profile screen so let's see in the console there is this updated user and
2:56:06
in the image property you can see there is the path and this is the data of our
2:56:11
file upload response we have full path the ID and then the Path property so
2:56:18
hopefully the image got uploaded so let's see into the storage uh let's refresh this so here we have a new
2:56:26
folder profiles and if I click on this we have a new file and this is the file that we
2:56:34
uploaded so it got uploaded successfully so let's see into to the users table if
2:56:41
it's uploaded there then hopefully we'll get a path here and yes we do so here is
2:56:46
the path for for that file so now we can use this to get the image and then show
2:56:53
it into our profile section in here so uh let's just minimize
2:57:00
this and there is a console log into the upload for file function so let's just
2:57:05
remove this and let's save this so the image is updated into the all user State
2:57:13
and we can use this to display here so this is how we are displaying the gallery image image and if we go into
2:57:19
this function this is where we need to use the superbase file URL for the images so let's just copy the URL for
2:57:27
this uploaded image and uh we need to return this URI
2:57:32
here but uh let's create a new function that will return the superbase file
2:57:37
URL and let's call it get superbase file
2:57:43
URL and this will receive a file path and it's the same path that we stored into the users table so we'll check if
2:57:50
we have the file path in that case we will return an object with a URI and we'll just paste the copied uh URL here
2:57:59
and if we don't have a file path we'll just simply return null so in this URL at the end you will
2:58:06
see uh we have the file path that we stored into the users table so profiles
2:58:12
slash uh the file so you can see this into the image that we have in the user
2:58:18
so we need to replace this with the file path that we passed to this function so let's use file path
2:58:26
here and if you notice uh we have this string at at the starting this is the
2:58:32
same string as the project URL so if I go into the constants you will see superbase URL it's the same string so we
2:58:40
can just replace this so let's remove this and use the superbase URL here and
2:58:48
this will work the same now let's use this function and uh let's just return
2:58:54
this function here and we need to pass the image path to this
2:59:00
function and if I save this now this will get us the remote file that we uploaded and we can see this into the
2:59:07
profile section because we use this function in the profile screen so let's go into the edit screen and here uh in
2:59:15
this uh code basically this will show the uh the local image if we pick from
2:59:21
the gallery but if we don't have the local image then this function uh will
2:59:26
return us the superbase file if we have any otherwise it will just return the default user image so let's go into the
2:59:34
edit profile screen and see if the local image is still working currently it's
2:59:40
showing the image that we uploaded and if I choose a new image from the gallery
2:59:45
let's choose this one and now it will show the local image from the gallery so everything seems to
2:59:52
be working all right now let's go to the profile screen and I think we don't need
2:59:57
to change anything here so here we using the Avatar component and we are passing the user.
3:00:05
image to this component and inside this component we are using the function as get user image source and this is
3:00:12
already set up so we don't need to change anything here now let's try to update uh the user details without
3:00:19
uploading the image and see if this is working so I'll just update this to um
3:00:27
updated then hit update so the name is updated and we can
3:00:32
still see the image so now we can just remove this updated
3:00:38
text and now let's try to upload a new image for this
3:00:43
profile um let's use this one and then hit
3:00:49
update give it a second there we go so we can see the new
3:00:55
profile image and even if I refresh the app you will still see this new profile
3:01:00
image so let's go to our document and see our progress uh we have implemented the
Create Post
3:01:07
update functionality now we're going to work on the new post screen so uh let's create a mockup for this
3:01:15
screen so it's going to be very similar to to the Facebook create post page so
3:01:20
let me show you how it will look so first we will have a back button on the
3:01:25
top left corner here and then we will have the title of the page which will be create post this will be shown in the
3:01:33
center we will use our header component for this then uh we will have a user icon on the left
3:01:39
side and the username as well so username this will show with the user
3:01:46
icon after this we will have a text area for the post body and we will Implement
3:01:52
a re text editor for this we will have all the options to style the post body
3:01:58
then uh we will have an option to add media to your post so we will have a a
3:02:04
box here and here uh we will have two icons for the image and for the
3:02:11
video so let's add them here if user clicks on the video we will show only
3:02:17
the videos from the Gallery if image then we will show only the images and we will have a text here add media to your
3:02:24
post then finally we will have a button here to submit the post and this will say post like this and let's match it to
3:02:33
our theme using this screen color so this is how our create post screen will
3:02:38
look like so let me just create a label here and the first step is to use the
3:02:43
header and we already have a component for this so let's use this
3:02:50
so let's go to the new post screen and here first we're going to import our screen
3:02:58
wrapper then inside this uh let's just remove this and we will import our header component and give this a title
3:03:05
of create post like this and let's save this so
3:03:14
there we can see the header but we need to have a pedding around the container and before that let me just copy and
3:03:20
paste the styles for this whole component and I'll show you whenever I use these Styles so we'll also need to
3:03:27
import the wp and HP functions and let's also import our
3:03:34
theme object okay so everything's good to go
3:03:39
now let's uh give this screen wrapper a background of white then let's create a
3:03:44
view and give this a style of container
3:03:50
and let's move this header inside
3:03:56
this now let's add a scroll View and we're going to give a gap
3:04:02
between all the items so let's use content container style and give it a gap of 20
3:04:08
pixels and inside this we will create our header section and in which we will create our Avatar so let's create a view
3:04:15
give this a style of header and let's save this so now for the
3:04:22
header we have the following Styles we are using Flex direction as row because we will have the Avatar and then the
3:04:28
username in a row so let's use our Avatar component and for using the user
3:04:34
image we will need to uh use the user data from use Au
3:04:39
hook like this now let's give this a urri of user. image
3:04:48
then for the size we will use HP of uh let's say
3:04:54
6.5 then uh we will use our rounded property to make it rounded and let's use theme. radius.
3:05:03
excl let's close this and save this so there we can see the user Avatar now for
3:05:09
the username let's create a view and give this a style of Gap 2
3:05:15
pixels and give this a style of
3:05:21
username and here we'll just simply use the user object to display the
3:05:26
username like this so we can see the usernames and here is the username style
3:05:33
uh you can just copy these Styles now I'm just going to copy this text for a public text this will show at
3:05:40
the bottom and we're not making functional but by default because all of the post are public so this will be a
3:05:48
static text as public like this and these are the styles for the
3:05:53
public text so you can just copy them from here then after this we're going to
3:05:58
create a view that will show our text editor so after this view let's create a new view and give this a style of text
3:06:08
editor and inside this we're going to create a new component that will show our editor so let's create a new file as
3:06:15
R text editor then create a functional component
3:06:21
inside this let's save this and let's just import the component
3:06:27
here and let's save this so there we can see the is text editor and for the text
3:06:34
editor style uh we don't have any Styles so you can just ignore this now we're
3:06:39
going to implement the Rich Text Editor and for that we're going to use a library called react native pel Rich
3:06:45
editor it's a very great Library so let's just copy this command to install
3:06:50
this quickly I will also need to install react native web view so let's close our
3:06:56
server and install this library after that we also going to
3:07:02
install this dependency react native web view so let's copy this and here you can see an example of how we can use this
3:07:09
library and Implement our each text editor and you can find the documentation for all the features in
3:07:16
this Library so it's installed now let's install react native web
3:07:27
view okay so now let's try to run our
3:07:35
project let's
3:07:40
reload okay so we don't see any error that's a good sign and uh now uh let me
3:07:46
show you a working demo of this Library so if you go to their GitHub repo uh
3:07:52
let's go into the examples and inside the source folder
3:07:57
you'll find this example. TSX file here it's a fully working re text editor with
3:08:03
all the possible options and we're going to use the components like uh R toolbar
3:08:08
here is the component and we're going to use the editor as well then we can also use this R toolbar
3:08:17
with the keyboard avoiding view with all of these actions and this will show on top of the keyboard but in our app we
3:08:24
don't have a situation like this so we don't need to implement this so I'll use these components to create the fully
3:08:31
working AR text editor from scratch and if you're confused about any property you can always find the documentation
3:08:38
here so now let's use this editor into our application and now we need to create a reference for this editor and
3:08:46
also for the content of the this editor so let's go to new post and here let's just create everything that we need so
3:08:52
body reference uh let's use use ref hook and then we will have an editor
3:08:59
reference for the editor and by default it will be null then we are going to use router to
3:09:07
move back to home screen when the post is created and let's use the loading
3:09:19
we will also upload the images and the videos so let's create a file state for
3:09:26
them like this now let's use this editor reference and pause it to our
3:09:34
component and we will also pause an on change function that will change the body reference so I will call this
3:09:41
inside the component this will give us a body and we'll just change the body reference. value to like this and we
3:09:50
will store all the content into the body reference now let's go inside and receive these properties editor
3:09:57
reference and then on change function so now I want to add a fixed
3:10:03
minimum height for this editor but obviously this will expand when we have more content so let's just give it a
3:10:10
style of minimum height of 285 pixels now let's remove this text and
3:10:18
here we need to implement the rich toolbar so let's just import this component Rich
3:10:25
toolbar this one and here we need to give a property as actions so this will
3:10:30
be an array of all the actions that we can use to style the content of our editor and I can show you into the
3:10:38
[Music] documentation so here is the list of all the common actions that we can use to
3:10:44
set the text as bold italic or in a bullet list you can also use this
3:10:49
documentation to render your own custom actions so let me just copy these actions and just paste them
3:10:57
here and we will also need to import the actions from this Library like this so
3:11:04
we will use these actions so now uh let's add the style for this as styles.
3:11:11
richar then for the container Styles let's use flat container style and give
3:11:17
the this uh styles do list style and we also want to connect it to
3:11:23
an editor and for that we will need to use the data reference like this and let's set the disabled property as
3:11:31
false and let's close this and uh let's indent
3:11:41
this um let's just indent this like this and
3:11:46
let's save this okay so we can already see all the
3:11:52
actions in the this toolbar and we have a warning at the bottom uh toolbar has no editor that's true we don't have an
3:11:59
editor yet this is just a toolbar so we can see all these actions now I want to
3:12:04
add custom actions so we can uh these are not the custom actions but uh we
3:12:09
don't have the icon for these actions so I want to add heading one and heading
3:12:15
four and if I save this you will see the library doesn't have the icons for them
3:12:21
and we can use our custom text or icon and for that we're going to use the icon
3:12:26
map property so we can map any action to uh a view that we want so there you can
3:12:33
find the documentation for this and let's use this to map the actions for
3:12:40
heading one and heading four so here we can just provide actions. heading one
3:12:47
like this and this will give us a call back with the tint color of the
3:12:54
icon and we will use this to uh set our style as active so let's ose a text
3:13:00
component and give this a style of tint color and inside this we'll just simply
3:13:06
add H1 so let's save this and there we can see the heading one icon so let's copy
3:13:12
this for the heading four and similarly you can add all the icons for other
3:13:18
actions as well so let's save this and uh we need to add a comma between these two so now we can see heading one and
3:13:25
heading four so this is how you can add the icons or map the icons to uh any
3:13:31
action so let's add a style for the rich bar and let's give it a border top right
3:13:40
radius of theme. radius.
3:13:45
excl and let's use use the same for the b top left radius because we want to make uh it
3:13:53
rounded on the top corner and let's give it a background color of theme do color.
3:13:58
gray let's save this and after this we will add our editor but now I want to
3:14:04
change the active color for these action so whenever these are active this will show the green color from our theme and
3:14:11
for that I can use a property selected icon tint color and let show the theme do colors.
3:14:19
primary dark color and I think this won't work now
3:14:24
but when we have the editor this will work so let's just save this and uh now let's add our editor let's import the
3:14:32
Rich Text Editor from this library and let's give this a reference for the
3:14:40
editor and let's add the content container style as the styles.
3:14:49
Rich then we want to add the editor Styles as well so these are the styles for the content inside the
3:14:56
editor so styles. content style then we want to have a placeholder
3:15:03
for this editor which will be uh what's on your
3:15:10
mind then uh finally on change property will trigger the on change function that
3:15:16
we paused from the parent component so here is the on change function so let's just close this and let's save
3:15:24
this and we have an error so I know about this error and I'll show you how you can fix this but for now let's just
3:15:31
dismiss this and we can see we have an editor H we have a placeholder and if I
3:15:36
type anything here I can able to make it board or italic first I will need to
3:15:42
select it then I can make it board italic and it's working I can remove the
3:15:47
Styles then I can uh align it to the center to the right or left and I can
3:15:54
make it in codes so all the actions are working now let's just refresh the app
3:16:00
and see if this error goes away so we still see this error but
3:16:06
don't worry I'll show you how you can fix this now let's add the style for this editor and make it look good so
3:16:13
let's add the styles for rich and I want to have a minimum height of
3:16:19
240 pixels then a flex one border width of 1.5
3:16:28
pixels and I don't want to have the Border top so let's use border top width
3:16:33
as zero then uh we want to make it rounded from the bottom so let's use
3:16:39
border bottom left radius as theme. radius. xcl then same for the Border B
3:16:47
right radius so let's just change this to right
3:16:52
radius and then we want to have a border color so let's use the gray color from
3:16:57
our theme and we want to have a ping of five
3:17:04
pixel like this and let's save this so there we can see the uh the container
3:17:10
style for our editor now let's add the style for the content inside the container so content Style
3:17:17
as the color will be the text dog color from our
3:17:23
theme and for the placeholder text we will use the gray color so placeholder
3:17:30
color will be gray and let's save this so and now I want to add the styles for
3:17:37
the list of the toolbar actions so we have the list style so let's just change
3:17:43
this to flat style and we will specify the styles for this
3:17:49
so I want to have a bedding around the list so I will add the bedding horizontal so let's use padding
3:17:56
horizontal of 8 pixels and let's save this so now we can see we have some
3:18:01
padding around the list we can increase the spacing so let's use 20 and we'll
3:18:08
have more spacing let's just use eight8 is fine then we want to have a gap between these items so let's use five
3:18:15
pixels uh which is to much so let's use three now this is fine so with this our
3:18:21
editor design is complete and uh it's looking good now I want to see if I add
3:18:27
more content will it expand so let's move to next lines and yes it's working
3:18:33
so let's just remove this and see if all the actions are working so I will add a
3:18:39
text as code with noi then I want to make it Bard and I
3:18:46
can make it to Center or to the left I can make it H1 and then move it around
3:18:53
like this then I can choose this code icon this will turn the text into code
3:18:58
snippet like this so let's remove this and see the number list so if I click on
3:19:05
this I can make a list and the text is bold so I can turn it off for the next
3:19:10
item we don't have a bold text so everything is working and all the actions are working so let's just uh
3:19:17
refresh the app and now I want to fix this error so
3:19:25
the error is because we are using this library on the latest SDK 51 from Expo
3:19:32
so if I go to the issues on their GitHub repo you will see editor not working on
3:19:37
Expo STK 51 and uh we already have a solution for this thanks to Goat iio so
3:19:45
inside the rich editor file we need to remove this data detector Types on the line
3:19:51
267 so if you don't see this error you don't have to do anything you can just
3:19:56
skip this part but if you see this error then we will have to patch this so let's go into note modules and find this
3:20:07
Library um so this is the library let's go into the source folder then
3:20:15
editor then here we need need to find the data detected types property on the
3:20:20
line 167 or no it was on
3:20:27
267 so here we can see this is the data property that was causing this issue so
3:20:33
we need to remove this and hopefully our issue will be fixed so let's just save this and restart our server so these
3:20:42
changes are not permanent we will lose these changes if we uh if we run the npm
3:20:47
install command this will override all the library code so we need to make a
3:20:53
patch so I'll show you how you can patch these changes so now we can see the
3:20:58
error is gone so everything's good to go now we want to use this patch package to
3:21:04
uh make these changes permanent so let's just copy this command and run into our
3:21:10
terminal and let's close the server again so this Library basically creates
3:21:15
a patch file that will change the library code whenever we install all the
3:21:21
packages so we will need to use this command and we will also need to copy this script Let's uh go into our
3:21:29
package.json and we need to paste this command into
3:21:35
our scripts this will run every time we run the npm install command and this will patch the changes so we are not
3:21:42
using yarn so we will need to use this command so let's just copy
3:21:49
this and after that we will need to specify the package that we want to uh
3:21:55
patch so uh let's use the name of this package and paste it
3:22:01
here now it will create a patch file using the changes that we done into the
3:22:07
r editor file and uh it's done it's created a patch file so it should have
3:22:13
created a patches folder um let me just remove the git from this folder so if
3:22:18
you want to remove git just type this command and whenever you save a new file
3:22:25
so let's just save this uh the git will be removed so here let's see we have a
3:22:31
batches folder here and here we have a new patch file so this code will make the changes to this Library whenever we
3:22:38
run the npm install command and even if we override the changes when we install
3:22:44
all the packages this will share make sure the patch persists so let's run our
3:22:51
application and we can just remove this file this one and this one too now let's
3:23:00
refresh and we won't see this error now so let's go to the new post screen so
3:23:07
everything is working now let's continue our design and let's go to the new post
3:23:14
page now I want to add the media section at the bottom of this editor so let's
3:23:20
add a view and give this a style of media from our
3:23:25
stylesheet let's close this and save this you can already see the Styles so
3:23:31
these are the styles for the media container we are using Flex direction as row because I want to add a text
3:23:38
property on the left and the icons on the right side so let's add the icons
3:23:44
first I want to add the text let's add add the styles for the add image
3:23:50
text and this will say add to your
3:23:58
post then after this we want to have a view that will contain the icons so
3:24:03
let's create a view give this a style of media
3:24:11
icons and if I show you the Styles we have the flex direction as row so you
3:24:16
can just copy these Styles and inside this let's create a touchable
3:24:22
opacity and now inside this I want to use the icon so let's use our icon
3:24:28
component and give this a name of image size will be
3:24:34
30 and the color will be uh the dark color from our
3:24:39
theme let's close this and save this so now we can see the image icon so let's
3:24:46
Al Al add an on press method and this will trigger an on pick function which
3:24:51
will open the image gallery so let's create this
3:24:57
function this will be an A String function and I also want to add a parameter as the is image so if this is
3:25:06
true that means we we need to open the gallery with only the images if not then we will open the gallery with only the
3:25:13
video files so for this one it will be true and it will show only the images so
3:25:18
let's just copy this for the video icon and let's change this to
3:25:23
false and I also want to change the name to video icon let's save this now we can
3:25:29
see the icons um let's increase the size for the video icon to 33 now it looks
3:25:36
good so we'll work on the onp pick function later but now I want to add a button at the bottom this will be after
3:25:43
the scroll view so let's import our button component and let's give this a button
3:25:50
style property and let's give it a height of
3:25:57
HP 6.2 then we going to have a title as
3:26:06
post let's add the loading State as well so we don't want to have the shadow
3:26:13
for this button so has Shadow property will be false and on press method will trigger an
3:26:19
onsubmit function which will create in a second so let's create this on submit
3:26:26
function this will be an async function and let's save this so there we
3:26:33
can see the post button at the bottom and it's not scrollable because it's out of the scroll view so now we need to
3:26:40
open the image gallery but that is already implemented inside the edit profile page so so let's just copy uh
3:26:48
this code and this will open the image picker we also need to import the image
3:26:54
Pier Library so let's copy this import statement and paste it here so by default it will open the image gallery
3:27:01
and show only the images but I want to make the configuration Dynamic based on the is image property so let's create
3:27:08
media configuration and let's just copy this configuration here so this will be the
3:27:16
default configuration and let's remove this and use the media config here now I
3:27:22
want to make an if condition if the is image property is false then we want to
3:27:28
show only the videos inside the gallery so uh let's change the media configuration and have the media type as
3:27:36
image picker let's just copy this one and change this to videos now this will only show the
3:27:43
videos and I also want to add the editing option so allow editing
3:27:49
true and this will get us the result then after we get the result we want to
3:27:54
set this file into this file state so uh let's see if the result is not
3:28:04
cancelled then we want to set the file and we have an assets array but we want
3:28:10
to save only the first object at zero so let's save this and hopefully this will
3:28:16
work so if I click on the image icon we see only the images and if I cancel it
3:28:22
and open the video now we see only the video so this is working now uh let's
3:28:27
choose an image um let's choose this one and hit
3:28:33
choose so the image that we picked is set into the file state but we are not
3:28:38
able to see it so let's show it under the Rich Text Editor so let's have a
3:28:44
condition here if we have the file then we will show this view so let's create a
3:28:50
view and give this a style of
3:28:56
file and let's save this so we can already see the blank space for the file
3:29:02
view and we have the following styles for the file we have the height of 30% and you can copy these styles for the
3:29:08
file so now I want to create a function that will return the type of the file so
3:29:14
let's use get file type and we need to pass the file and this will return
3:29:20
either video or image we're doing this because I want to show a video player if
3:29:25
the file is video and if the file is image we will simply use the the image
3:29:30
component so for now let's just add an empty tag
3:29:36
here now let's create this function get file
3:29:42
type and this will receive the file and first we will need to make make a condition if we don't have a file then
3:29:48
we'll just simply return null then we need to make one more
3:29:55
condition if the file is local or remote so let's create a function is local
3:30:00
file um let's create it here this will also receive the
3:30:07
file and let's also make a condition here if we don't have a file then we'll simply return
3:30:14
null but if we do have a file then we will make one more condition and check the type of the file and if it equals to
3:30:22
an object that means uh we have a file object and we picked it from the local Gallery so we will return true otherwise
3:30:30
let's return false now let's pass the file here and if it returns true that means we have a local file and for every
3:30:37
local file we have a type so whenever we set the file here we will have a type
3:30:43
property on this file and this will be either video or image so we will return that so this will handle the local files
3:30:51
but we also want to check for the remote files because later I want to use this page for updating the post as well so we
3:30:58
will need to uh make a check for the remote files so let's create a comment here
3:31:04
check image or video for remote file and in our superbas storage we will have two
3:31:11
folders for post videos and post images so every file will will contain a path
3:31:17
as post image or post video so we will just make a check here if the file includes post image that means it's
3:31:25
saved into the post images folder and we have an image file so we will return the
3:31:30
type as image but if it's not then we will just return
3:31:37
video like this so this function will return the type of the file either
3:31:42
remote or local now I want to show you the type property on the local file so
3:31:47
let's just copy this and console log the file
3:31:52
here and let's save this so now if I open the image from the gallery let's
3:31:58
choose this you will see in the console um here you can see we have a property
3:32:04
type as image so this is what I was talking about and if we choose the video
3:32:09
we will have a type as video so here you can see we have the type video
3:32:16
so let's just remove this console log and this function will give us the
3:32:24
type of the image uh we will only get the type for the local files for now but it should hopefully work for the remote
3:32:30
files later so let's use this and here if it's not a video then it's an image so let's
3:32:38
return an image component and let's give this a source
3:32:45
of an object with a URI so we will need to create a function that will get the
3:32:51
file URI uh this will check if the file is local or remote then it will return
3:32:56
the file URI then resize mode as cover and the style uh let's choose Flex
3:33:03
as one let's close this okay so now we need to create this
3:33:09
function get file URI this will receive the file and first
3:33:16
we'll need to make a condition if we don't have a file then we will just simply return
3:33:23
null and let's also make another condition for if the file is local so we
3:33:28
will use our function is local file if it's true then we already have the URI
3:33:33
property on the file we will return that but if it's a remote file then we will
3:33:38
need to return uh the function get superbase file URL we already have that
3:33:44
in our image service so let's import this get superbas file URL and we need
3:33:49
to pause the file so if I go into the get superbase file URL this returns an object with a
3:33:57
URI but uh we only need the URI so we will just simply uh return the URI from
3:34:04
this function like this so this function will return the file path if it's local
3:34:10
or remote so let's see if this works let's choose an image
3:34:18
and we should see this in here but we don't see it and I think I made a mistake
3:34:25
somewhere so resize mode cover Flex one and get the file
3:34:33
uuri so this will check uh this will return null if local file then file
3:34:41
URI so everything okay all right but uh
3:34:47
let's just try to console log the file and see if we are getting the file URI so we'll just call the get file URI
3:34:54
function here and pass the current
3:35:03
file so we're getting the file URI from this
3:35:08
function so this is working so there must be something wrong with our image
3:35:13
component and oh okay so this should be Source not Rous so this was causing the issue and
3:35:21
now we can see this image and it showing from the local Gallery so now if I
3:35:26
choose another image it will show that let's show this SC
3:35:32
image and it's updated and if we choose another this will be updated as well so
3:35:38
now I want to have a way to delete this basically to reset the file state so
3:35:43
let's create a pressable here and inside this we will uh import our
3:35:50
icon component and use the delad icon let's give this a size of
3:35:57
25 and the color will be
3:36:02
white so let's save this and uh it's white so we not able to see it so let's
3:36:08
give it a style of close icon let's save this and we can see it
3:36:15
on top of the image so for the size let's use 22 now this looks good so for the close
3:36:23
icon we have the following Styles it's positioned absolute on the top right corner now I want to have a background
3:36:31
color of transparent red so let's use rgba colors and I use the opacity as
3:36:39
0.6 uh currently it's black so let's increase the red color
3:36:46
and I think we need to have some bedding so let's use bedding of five pixels and we need to make it rounded so
3:36:54
let's use B radius of 50 pixels um let's increase the padding to
3:37:01
S and we need to make it a bit smaller so let's change the
3:37:08
size um here it is so let's change the size to 20 let's save this now it looks good so now when we click on it it
3:37:16
should reset the file state so let's add an onpress method and here we'll just
3:37:21
simply set the file as null and save this and now if I click on
3:37:26
this it removes a file so let's open another file we see the file and if we click on
3:37:33
this icon it will set the file as null So currently we only have the image
3:37:39
component but now I want to show the videos as well so if I choose the video we will only see the style but we don't
3:37:46
see the video so for videos we need to install another Library it's called Expo
3:37:52
AV it has great features for audio and video files so let's just copy this command and quickly install this let's
3:38:01
open the terminal and paste it
3:38:08
here okay so it's installed now let's just close this and we need to import
3:38:13
the video component from this Library so let's remove this and let's import the
3:38:19
video um I think we will need to manually import this so let's import the
3:38:26
video component from Expo a this
3:38:33
library and let's use this here so we need to provide style as Flex one and
3:38:41
Source will be the same as the image component as an object with URI and we
3:38:46
will use the function get file URI let's pass the file
3:38:51
object then we need to have a property use native controls then resize mode as
3:38:58
cover and we need to add a sloping property so this will Loop the video and
3:39:04
let's close this and save this so as we save this you can see we
3:39:09
already have our video player and it's working I can play and pause the video I can move forward
3:39:15
back so everything is working and if I remove this the video is removed now
3:39:21
let's select this again and let me show you this cool feature so before selecting the video we can trim the
3:39:27
video and then choose now only the trimmed part of the video is selected and then I can also perform all these
3:39:34
actions I can play pause or make it full screen so everything is working and with
3:39:40
this our media section is complete now I want to work on creating the post into the Super base so for that we will need
3:39:47
to work on our onsubmit function and here first we'll need to have the post
3:39:52
body and the file and we are already storing all the content into the body
3:39:58
reference so let's console log this inside this function and see uh if this
3:40:03
is actually updating and we'll get the content
3:40:10
inside the current property of this reference then let's also console log the file
3:40:17
let's save this and now if I click on post uh we will see in the console as
3:40:23
the body is empty and we have the file so let's add the content hey what's
3:40:29
up and let's post now we see the body as the HTML mockup and we have the file
3:40:35
object here so let's make a condition here so that user will be able to post
3:40:41
only the content or the file or both of them at the same same time so if either
3:40:47
of them is uh not added then we'll simply prompt the user to add an image
3:40:53
or the post body so let's choose alert and title will be post and we want to
3:40:59
show a message please choose an image or add the post body then after this prompt
3:41:07
we'll just return from here and if we have either one of these values then we
3:41:12
will create the post data so this will in Lo the file we will upload this
3:41:17
inside the create post function then for the post body we will use body reference. current value and we will
3:41:25
have the user ID who created this post so let's use the current user.
3:41:31
ID um I don't think we have the current user here in this file we have the user
3:41:37
so let's use the user. ID here so this will be the post data so
3:41:44
now we want want to create a create post function but I want to make it generic
3:41:49
so that we can use the same function for updating the post as well so let's just make a comment here for now and let's
3:41:56
create our post
3:42:02
service and inside this we need to export a function create or update
3:42:08
post and this will be an async function which will receive the post add up and
3:42:14
let's create a Troy catch block inside this and if we catch any
3:42:19
error we will console log the error as create post
3:42:25
error and in that case we will return our response as success false and we
3:42:32
will include a message as could not create your
3:42:39
post like this so now first I want to upload the post file so let's create a
3:42:45
comment upload image and first we need to make a check if we have the file
3:42:50
object inside the post then we will upload the file let's also add a
3:42:55
condition if the type of the file is an object that means it's a local file and
3:43:01
user picked it from the gallery and it can be either a video or an
3:43:07
image so when this is true we will uh check if it's an image or not so is
3:43:13
image and if it's a local file file it will also include a typee property and this will be either image or video so
3:43:20
we'll just make a check if it's image then uh is image will be true otherwise false and based on this is image
3:43:27
property we will create a folder name so if it's an image we will use the folder as post images and if it's a video then
3:43:35
the folder name will be post videos we're also using this folder name when we are checking for the remote files if
3:43:41
the file is video or image so if you see here file includes post image and the post image is basically the folder name
3:43:48
that's how we know if the file is image or not so let's update these two post
3:43:54
images like this so let's go back and here we need the folder name for
3:43:59
uploading the file into super base so let's use the upload file function here
3:44:05
and this will give us a file result let's use await file
3:44:12
upload and this requires three parameters folder name we already have that here then file URI is on post.
3:44:19
file. U every media object has this URI property then let's pause the is image
3:44:25
property to check the file inside the upload function and this will give us
3:44:30
the result and we will check if the success is true on this response that means the file is uploaded and we have
3:44:37
the data as the path of this file so we'll just update post. file to this
3:44:43
data otherwise if the the file is not uploaded we will return the file result and this will show the user that the
3:44:49
media was not uploaded so if I go into the upload file here you can see on
3:44:56
successful upload it will give us the path of the file here and if it's not
3:45:01
uploaded we will see could not upload media so everything's good now after
3:45:06
this if statement we need to create the post and we will get the response as
3:45:12
data and error now let's import our supervis client and we want to upload this data
3:45:20
into the post stable so from Posts and we need to call a function
3:45:27
upsert and it will perform update and insertion as well so if I show you the
3:45:33
documentation from the super base um let's go into database and
3:45:40
upsert data so this is the Syntax for this function and we can include in all
3:45:45
these options so when we pass the data to this function if it doesn't include
3:45:50
the ID this will create a new row into the table but if it does include the ID
3:45:55
that will mean it will update the row with this ID now let's use the select
3:46:01
function to select the data after insert or update and this will give us an array
3:46:06
so let's use the single function to get the single object now let's check if we
3:46:12
have an error then we will console log the error create post
3:46:17
error actually let's just copy this response because it will be the same
3:46:23
response and if we don't have any error that means the post got successfully
3:46:28
updated or created and in that case we will return success as true and data
3:46:34
will be the data from this response so let's save this and now we
3:46:40
can use this inside our create post screen and and let's go to on submit
3:46:47
function and here first we need to start the loading so let's set the loading to
3:46:53
true then we'll get the response from this new function so let's use create or
3:46:59
update post and we need to pass the data then we will need to set the
3:47:05
loading to false after the API call is done and we'll just simply console log
3:47:11
to see the post result as as this response let's save this and test if
3:47:18
this is working so let me just refresh the app and open the terminal um let's remove this user
3:47:25
console log so let's go into home screen and here is the console log let's command it out and let's
3:47:33
refresh so now let's choose an image um let's choose this beach
3:47:41
image and let's add a caption
3:47:46
then hit post so we see the loading and then we
3:47:51
see the post response in the terminal so we got the data and success as true so
3:47:57
our post got uploaded successfully so let's go into the super base and see
3:48:02
into the post table so here we can see our first post
3:48:08
as the ID body file and the user ID so you see this file has the post images SL
3:48:14
the file name so it must be uploaded into the superware storage so let's go into the storage uploads then we see a
3:48:22
new folder post images and we have a file inside this so this is the file
3:48:28
that we uploaded so now the post create is working and the media upload is working
3:48:34
as well now I want to clear this data when the post creation or updation is
3:48:40
successful so let's go into the new post screen
3:48:46
and let's remove this console log and here we need to make a check if response. success is true then we clear
3:48:52
this data but if it's not then we need to show an
3:48:58
alert let's use the title as post and for the message we will show response.
3:49:04
message so here if the post is created we will set the file as
3:49:10
null then we also need to uh set the body reference do current value to an
3:49:15
empty string but this will not clear this editor content so we need to set
3:49:21
this editor content to an empty string as well so editor reference. current and
3:49:27
we can use this function set content HTML this will set the content inside
3:49:32
this editor so we need to set it as an empty string then after this we need to
3:49:38
move back to the previous screen so router. back this will take us to the
3:49:43
previous screen so let's save this now let's just refresh the app and see if this is working so let's create a
3:49:50
new post and this time we want to upload a video so let's choose a video and
3:49:56
let's actually trim this so we have the option to trim the video before
3:50:03
selecting so now we can also post the media without the caption as well so
3:50:08
let's just post this video and hit post so we move back to the home screen
3:50:15
that means it got uploaded so let's refresh the superb storage and we see a new folder post
3:50:22
videos and if we click on it we have a new file and this is the file that we
3:50:27
uploaded so it's very important to choose the right path for your files and the right content type for your files so
3:50:35
let me show you how it's actually working so if I go into the create post function here I'm dynamically choosing
3:50:41
the folder name based on the type of the file then if I go inside this upload file
3:50:47
function here I'm setting the video type for the content type so if you don't set
3:50:53
this the file will be uploaded but you won't be able to uh use this inside your application or you won't be able to play
3:51:00
this so it's very important you implement all of this very carefully so
3:51:06
now let's try to create a post without the media and see uh if we can create the post with only the caption so let's
3:51:13
create a caption hey everyone I new
3:51:19
here and let's also style this and I want to make this new text board so
3:51:25
let's just select this and hit board icon so now let's
3:51:31
post and we move to home screen so let's see this post into the super base let's
3:51:37
go into the table editor and go into the post table so here we can see all of the post
3:51:44
and we have a third new post where we can see the post body as the HTML and we
3:51:50
see the file as null because we did not include the file for this post so that
3:51:56
means we can create the post with or without the media we can also create the post with both the media and the caption
3:52:02
as well so with this the post creation feature is complete so now let's see what we have
Show Posts
3:52:10
done so far let's go to our document and here in New post screen we added the
3:52:16
header user Avatar then we implemented the editor add post uh we added super B
3:52:22
storage media and then we implemented the file upload using the Bas 64 so now
3:52:29
we need to work on our home section and fetch and show all the posts and before
3:52:35
we start working on fetching the post we need to create a mockup of the post card
3:52:41
just to see how all the post will look like in the home screen so let's create a mockup of the post card let's create a
3:52:53
rectangle so this will be our post card let's choose a transparent
3:52:59
background uh like this one so let me add a label this will be our post
3:53:05
card so here first we will have the user image so let's create a user Avatar then
3:53:11
alongside with this user Avatar we will have the user name and the time of the
3:53:18
post so let's display them along with the user Avatar so this will be a small
3:53:23
text and this will display under the username like this then at the bottom we
3:53:29
will have an area to show the post body and any media inside the post so if we
3:53:35
have any image or the video inside the post this will display under the post
3:53:40
caption in here then we have some actions at the bottom so hard icon will
3:53:47
like the post or dislike the post then we have the comment icon to make a
3:53:52
comment or to see all the comments in a post and then we will have a share icon
3:53:57
to share this post content any file or or the caption so finally we will have
3:54:04
the three dots icon on the top right corner so basically this will open the post details and user can see the
3:54:12
comment list and even create the comment from from there so now we need to add
3:54:17
this to the home screen so let's just copy this and use it
3:54:22
here uh let me copy this one more time um uh let's move this content below
3:54:31
so that we have a bit more
3:54:37
space okay so after this post I want to show a loading State at the bottom of
3:54:43
all the post so let's use a loading icon and place it here so this will indicate
3:54:51
that there are more post being fetched from SOA base and uh once all the posts are fetched we will just display a
3:54:58
message no more post so first we're going to work on fetching the post from
3:55:03
super base then we'll work on this card so uh first let's create a state
3:55:09
here post and set post and this will be an empty
3:55:16
array then we're going to call a function get posts this will fetch the post from super base and we will Define
3:55:22
another function inside our post service and we'll call this function in a use
3:55:28
effect hook so whenever this home screen mounts we will fet the post so let's use
3:55:34
use effect and let's move it to the top here and we need to add an empty
3:55:40
dependency array then call this function and this will get us the post so now we
3:55:45
need to create a function fetch post inside our post service so let's go into that file and um let's just copy this
3:55:55
function and rename it to fetch post and this will receive a parameter
3:56:02
as limit by default it will be 10 so uh by default we will fetch 10 posts so
3:56:08
let's just remove this code and let's change this to fetch post error
3:56:16
and this message will say could not fetch the posts like this now we're going to call
3:56:24
the API to fetch the post and it will give us a data and an error on the
3:56:30
response so let's use super based client then uh from the post
3:56:37
table we're going to select everything from this table so let's use select and we can pause static just like in the skl
3:56:45
queres and this will fetch all the post data and I want to fetch the post in a
3:56:50
descending order using the creation date so let's use the order function and pass the column as created at then as a
3:56:58
second parameter we can pause an option ascending as false this will fetch the post in and descending order then we can
3:57:06
use the limit function to limit the posts being feted and we want to use the limit variable so this way we can get
3:57:13
the post data from post table now if we get any error uh let's make a condition
3:57:19
and we will return this error here let's just indent this and if we
3:57:25
don't get any error that means we got the post and in that case we will return success as true and the data will be the
3:57:34
data object so like this so let's save this and see if this function is working
3:57:39
so let's move to our home screen and here let's call this function we will get a response fesh post uh we don't
3:57:47
need to pass anything by default we will have the limit of 10 so got post
3:57:55
result let's save this and see if this is working and we can already see uh we have some log in the terminal got post
3:58:03
result and this includes the data and the success object so uh this is the
3:58:08
data we already saved into subas so let's just copy this and let me format this using adjacent
3:58:16
formatter let's paste this here and prettify so here you can see we have
3:58:22
three post we have the body created ad file ID and the user ID so these are the
3:58:27
post that we created previously so one thing you notice we have all the post
3:58:33
data and the user ID but this does not include the user data and we need that to show in the post card so we will need
3:58:40
to make a few changes so here when we are selecting the dat from the post table because we have a foreign key
3:58:47
relationship with the user so we can get the user data so static will make sure
3:58:52
to fetch the data from the post table then we can specify users table then all
3:58:58
the properties that we want to fetch for each user so I want to fetch ID name and
3:59:03
image so let's save this now we got another console l so let's just copy
3:59:09
this paste it here and make it pretty
3:59:14
um we're getting this error because now we have the users object array inside
3:59:20
each post but I want to get only the user object not users so we can rename
3:59:26
this as user like this now it will only get the user object so uh now we see
3:59:33
another console now let's uh see this user object inside the home screen because our format is not working for
3:59:41
this so let's console log the user from our first
3:59:47
post uh result. data. first item and then the user property and let's save
3:59:55
this so there we can see the user data and we can use this user data to show in
4:00:00
the post card so uh now let's just remove this console log and minimize
4:00:07
this terminal uh let's also remove this and here let's check if the result success
4:00:14
is true then we will set the post State as result. data now I also want to send
4:00:20
the limit parameter so let's create a variable it will be a global variable limit with the zero value and we will
4:00:27
increase it every time we call this get post function so limit equals limit +
4:00:33
10 and let's pause it to the fetch post function so the idea is whenever we hit
4:00:39
the bottom of the screen we will increase the limit and we will call this function and it will fetch more and more
4:00:45
posts so let's just console log it here fetching post as limit and there we can
4:00:51
see fetching 10 posts so now I want to show all the post in the home screen so
4:00:59
let's create a comment here post and let's import the flat list
4:01:06
component and for the data I want to use the post State and I show vertical
4:01:13
scroll bar indicators as false then content container style we'll have
4:01:19
styles. list style and for the list style we have some padding horizontally
4:01:24
and padding top so here you can just copy this style and then let's add the
4:01:30
key extractor property and we will have a call back with the item I want to use the item. ID
4:01:37
but let's convert it to a string like this then uh we will have the render
4:01:43
item property this will also give us a call back and we want to extract the
4:01:48
item and this will return a component post card which we'll create in a second
4:01:54
and we want to send some properties as item and then the current user will be
4:02:01
in the user from All State and we want to use the router
4:02:07
inside this component as well so let's just close this
4:02:14
and let's also close this flatlist component and now we need to create this
4:02:19
post card component so let's create a new file post card then let's create a functional
4:02:25
component and save this now let's import this here and save
4:02:32
this so there we can see we have three Post in in the post State and we are
4:02:38
displaying them here now let's uh go into this component and receive all the props we have the item the current
4:02:47
user and then we have the router now let's console log the item
4:02:53
and see if you're getting the post data so let's save this and see in the
4:02:59
console so there we can see we getting the three post items so let's just minimize this I also want to have a
4:03:06
shadow on this post card but this will be conditional based on this property has Shadow and by default it will be
4:03:13
true so if it's true we're going to use a different uh style object for the Shadows so let's create this Shadow
4:03:25
style and first we're going to have the shadow offset as the width will be zero and
4:03:33
height will be two then let's add the shadow
4:03:38
opacity as 0.06
4:03:44
let's also add the shadow radius S6 and uh the elevation property for Android
4:03:50
will be one so these are the shadow Styles now
4:03:56
let me just copy and paste the style sheet here for this screen and I'll just show you these uh Styles whenever I use
4:04:03
them now let me just import the theme the HP function and let's also import
4:04:09
the wp function from our common helpers Okay so so now let's give this a style
4:04:15
of container and we'll also use our shadow Styles based on a condition so uh if the
4:04:23
has Shadow property is true only then we will apply Shadow Styles and let's save this so because this is true by default
4:04:30
we can see the Shadows so for the container we have the following Styles Gap margin padding the borders so you
4:04:37
can just copy these Styles now inside this let's just remove this and have a
4:04:43
view and give this a style of
4:04:48
header so for the header we have Flex row because on the other side I want to
4:04:53
show Three Dots and on this side I want to show the user info so let's create a
4:04:59
view and give this a style of user info let's just add a comment here user info
4:05:04
and post time now inside this view I want to use our Avatar component so
4:05:10
let's import it and for the size let's use HP of
4:05:18
4.5 like this and then for the URI I want to use the item. user. image
4:05:26
because we have the user data on each poost item so uh the image will be the
4:05:31
user image then around it will be theme. radius. MD like this and let's save
4:05:39
this so there we can see the user avator for each post item then let's create a
4:05:45
view and give this a gap of two pixels and inside this we're going to show the
4:05:50
the username and the time of the post so let's give it a style of username and
4:05:56
inside this let's choose item. user. name like this and if I save this we can
4:06:03
see the username so if you see in the console each post item has this user
4:06:08
object because we added this while fetching the post data and because we have a relationship with the users table
4:06:15
so let's just copy this for the Post Time and this will have styles of post time and here we will display item.
4:06:23
created at property so uh we can see the date but it doesn't look good and we need to
4:06:29
format it and for that we're going to install another Library
4:06:36
moment and it's installed so let's just close this and use this to format our
4:06:42
date so let's let's create uh created at variable and inside this we will use the
4:06:48
moment Library let's just import it and let's give it item. created at property
4:06:55
then we're going to use the format function and we're going to use a format of Triple M and then the D so this will
4:07:03
format the date as the English month and the date day number so let's use this
4:07:08
and let's save this so there we can see the date June 19 which is the time of recording this
4:07:15
video now I want to show three dots icon on the right side so let's create a touchable opacity and inside this let's
4:07:23
use our icon component and give this a name of three dots
4:07:29
horizontal like this and let's use the size of HP
4:07:36
3.4 and let's use a stroke width of
4:07:41
three and we will use the text color from our theme object so theme. colors.
4:07:48
text and let's save this so there we can see the three dots icon when we click on
4:07:55
this icon it should open the post details so on press on this function we
4:08:00
will trigger the open post details function so let's create this function here and we will implement this later so
4:08:08
for now let's just leave it empty now I want to show the post body
4:08:13
after this view so let's create a comment here post body and
4:08:20
media so now let's create a view and give this a style of
4:08:27
content and inside this we're going to show the post body so let's create another view and give this a style of
4:08:34
post body so here let's use the text
4:08:40
component and inside this we will use item body and let's save this so there we can
4:08:47
see the post body and it's displaying the HTML content but we want to render
4:08:53
this HTML and for that I'm going to use another library and it's called react
4:08:59
native render HTML so this is the library we're going to use this will render our HTML body so
4:09:07
let's just copy this command and quickly install this Library
4:09:16
so while this is installing let's just copy the import statement for this Library so let's just copy this
4:09:23
line and paste it here okay so the library is installed
4:09:30
now let's just close this terminal and minimize this so now we only want to use this
4:09:38
Library when we have the post body so let's just remove this and make a condition here
4:09:44
item. body only then uh we will use this Library render
4:09:50
HTML and uh we're going to give it a property of content width which will be 100% so let's use WP 100 then we're
4:09:59
going to use the source which will be our HTML body so let's see how we can
4:10:04
use this so we need an object with HTML property then the content so let's add
4:10:11
an object HTML and then item. body like this and let's close it and save
4:10:19
it so now it's rendering the HTML post body and we can see the styles are
4:10:25
applied as well but we also see three warnings at the bottom which are sport
4:10:31
for the default props will be removed which is something this Library uses but
4:10:36
it's not affecting the functioning of this Library so we can just ignore them but this will appear every time we
4:10:43
refresh the app so let me just reload the
4:10:49
application so when we get the post we will see These Warnings so we will need to hide them so they don't show on our
4:10:57
device but they will still appear in the terminal and that's fine we can leave with that so let's just go into the
4:11:04
layout file and here we need to use the log box to ignore all these
4:11:10
logs so let's import log box from native and we're going to use a function ignore
4:11:16
logs and this receives an array of all the logs that we want to ignore but we don't have to copy and paste the
4:11:23
complete log message we can just copy until this column and paste it here and
4:11:28
this will be removed so uh let's copy all the logs and this is the second one so let's
4:11:36
just copy this and paste it here and let's also copy the third log
4:11:46
and paste it here now this will remove all of these three logs and uh they will
4:11:51
they won't show on the device again so if I reload the application uh we can still see them in the terminal but we
4:11:58
don't see them on the device so everything's good now let's go into our
4:12:04
postcard and here I want to add some styles for the content and we can use our property tag
4:12:11
Styles and let's use Tag Styles so now let's define our Styles
4:12:19
here let's create a constant tag Styles and we want to use our theme
4:12:25
style as the color and the size so for the D element we will have the color as
4:12:31
theme do colors. dark and for the size let's use HP of
4:12:41
1.75 so this style will be used by multiple text components so let's just copy this and create another object as
4:12:49
text style and paste this here now for the div element we will use this text
4:12:56
style for the paragraph tag we will also use this and for the order list tag we
4:13:02
will also use this text style now for the H1 and H4 tag we will just change
4:13:08
the color because if we change the font size uh this will also make our heading tag smaller so we'll just choose color
4:13:15
as theme. colors. dark and let's just copy this and change this to
4:13:23
H4 then let's save this so there we can see a slight difference in the color and the size of
4:13:30
the text components so similarly you can add other styles if you want to now I
4:13:35
want to show the post image here but first we need to check if we have the file and it's an image file so we can uh
4:13:42
use a function includes and see if this file includes a string as post images because each file
4:13:51
has a path and it should include the folder name and if you see in the super
4:13:56
base each file includes a folder name post images or post videos so if it if
4:14:02
it includes post images that means it's an image file and in that case we will
4:14:08
use the Expo image component here and let's give this a source
4:14:14
and because it's a remote file we can use get superbas file URL and pass this
4:14:19
file path here and inside this function you can see if we have the file po this
4:14:24
will return the super base URI then let's add the transition of 100
4:14:30
milliseconds and for the style let's give it a style of post
4:14:36
media and content fit property will be covered
4:14:44
so let's close this and uh for the styles of the post media we have the height of 40% border radius border curve
4:14:51
and the 100% width so you can just copy this so let's save this and there we can
4:14:58
see the post image that we uploaded so this second post is empty because we
4:15:03
uploaded a video in this post now we want to show the post video so let's
4:15:08
create a comment here post video and we want to use the video component from
4:15:14
Expo a but first let's make a check if we have the file and the file includes
4:15:21
post videos that means it it's uploaded into the post videos folder so it must
4:15:26
be a video file and in that case we will use the video component from Expo AV so
4:15:32
this is the component let's just import it and give this a style of post media
4:15:38
so this style has the height of 40% but we want to use the less height so let's
4:15:44
override the height and use HP of 30% for the videos and if you see this will
4:15:50
just override the height from the post media style so now let's add a source
4:15:56
and we can use the same function and get a supervis file URL and pause the item
4:16:01
do file then let's use use native controls property this will use the
4:16:07
native controls for this player resize mode as cover and is looping so this
4:16:13
will Loop now let's save this and there we can see the video that
4:16:19
we uploaded and we can play this video um it's not
4:16:25
working so I can't play or pause this video so let me just reload the application and check
4:16:31
again there we see the video and uh we still not able to play so let's just
4:16:38
delete this file maybe this file is corrupted for some reason uh let's delete this from our database and we
4:16:44
will upload again and check if this works so let's
4:16:49
reload and now we don't see this post and let's create a new post with the same file let's choose this
4:16:57
video and then hit
4:17:03
post so now we need to reload the application because uh it won't appear automatically
4:17:10
here there we can see the new post and and now if I hit play it still doesn't
4:17:17
work that's very strange um let's just close this
4:17:24
application and open it
4:17:32
again we can see the video and if I click on play now it's working so I
4:17:38
think there was something wrong with the Expo Go app but we just needed to restart it and everything's working now
4:17:44
we can pause play go back go forward and all the controls are working I can given
4:17:51
make it full screen and this is working so all the controls are working and the
4:17:56
video post are working now so now I want to add some actions at the bottom to
4:18:02
like and share this post so let's get a comment like comment and
4:18:08
share then let's create a view and give this a style of footer
4:18:15
and for the footer we have the style as Flex row so if I open this you can see
4:18:21
these Styles I want to have three different actions for this footer area
4:18:26
so let's create a view and give this a style of footer button and inside this we are going to
4:18:33
use touchable opacity for the button then let's use the icon component
4:18:39
and give it a name of heart and size will be
4:18:44
24 and for the color let's use the rose color from our theme object let's close
4:18:51
this and save this so there we can see the hard icon to like each post so this
4:18:58
color will change based on a condition and for now let's just create a Boolean
4:19:03
liked uh let's make it false and based on this uh we'll change the color for
4:19:08
this heart icon so if light is true uh then we'll use the was color but if it's
4:19:14
not then we'll use the text light color from our theme
4:19:21
object like this and let's save this so by default you can see it's using the
4:19:27
text light color and then we going to show the count likes count so let's create a text component give it a style
4:19:34
of count and then inside this we're going to show the length of all the likes so
4:19:42
later we're going to have a likes array and we're going to use the likes length but for now let's just create an empty
4:19:48
array likes and give it an empty aray like this and we will just simply
4:19:55
display the length of this array so this will be the count for all the likes then
4:20:01
let's just copy this button two times for the share and the comment icon so let's change this icon to
4:20:10
comment let's save this and we can see the
4:20:15
comment icon and for this one uh let's just remove this and add zero for now
4:20:21
and for the third one let's use the share icon like this and we don't want to see the count so let's just remove
4:20:28
this and let's save this so uh we are using this liked
4:20:33
property to change the color of the Hide icon so if I change this to
4:20:38
True um we shouldn't see it on all the icons and we copied this on the comment
4:20:45
and the share icon so let's just remove this we only want to use the text light color on the comment and the share icon
4:20:52
like this so This rose color is only applied on the stroke of this icon but we want to change the full color of this
4:21:00
icon so if I go into this icon file in here here you can see we have a property
4:21:07
fill as none but if I change this to a different color let's say blue uh this
4:21:13
will change the fill of this color so we can use this from the parent component because we are spreading all the props
4:21:19
here so we need to make uh this fill property on this component and this will
4:21:25
be conditional as well so if the light is true we will use the rose color from
4:21:30
our theme but if it's false then we will use the transparent color so now it's
4:21:36
showing the full heart icon with the red color and if I make this false we will
4:21:41
only see the outl L icon without the fill color like this so This postcard
4:21:47
component is complete for the home screen now I want to show a loading State at the bottom this will indicate
4:21:53
that we fetching more posts so let's go into the home component and here in the
4:21:58
flat list I want to use a property as list footer component and this will show
4:22:04
at the bottom of the list so let's create a view and give this a style of margin vertical of 30 pixels
4:22:13
then let's close this and inside this we will use our loading component so let's
4:22:19
just import this and save this so now if I scroll
4:22:24
down we see the loading State indicating that we fetching more data but if I
4:22:29
reload the application it will show on the top when we have no posts and I want to increase the margin when we have no
4:22:36
post so let's create a condition if the post do length is zero then we will have
4:22:42
a margin vertical of 200 pixels otherwise 30 so when we have the post it
4:22:47
will show like this and if we don't have the post at the start it will show like this so now let's go to our document and
4:22:54
see our progress and in the home screen section we have fetched the post we have
4:23:00
designed the postcard I want to work on post pation but before that uh let's
4:23:05
work on Real Time implementation and this will mean when we create a post
4:23:11
this should automatically we are inside our Post Feed in the home screen so for
4:23:16
this first we'll need to enable the real time changes let's go into the database
4:23:22
and then into Publications here we see the real time
4:23:28
superbase changes are enabled but uh we see the source as zero tables so let's
4:23:34
click on this and we need to enable the tables that we want to receive the real
4:23:39
time changes from so let's choose comments no notifications and then posts
4:23:45
so let's go back and these are the actions that we can listen to and we can even turn off an option and we won't be
4:23:51
able to listen to this on the Real Time Changes now I want to listen for the
4:23:56
create post event so I need to go into the home screen and inside the use
4:24:01
effect hook here I need to create a channel that will listen to the insert events on the post table so let's create
4:24:09
a post Channel and let's use super based client then uh let's use the channel
4:24:16
function and give it a name of posts this can be any name then I'm going to
4:24:21
use the on function to listen on the postest changes so let me show you how
4:24:27
you can use this from the documentation and we need to go into the
4:24:32
real time changes and click on the subscribe to
4:24:37
channel so this is the syntax to subscribe to a channel and then listen
4:24:42
all the events from that channel but in our case I want to listen to the insert events from the post table so let's go
4:24:50
to the listen to a specific table and this is how we can uh listen to all the events from the countryes table and this
4:24:57
will give us a payload every time we create lead or update anything inside the countries table so let's use this
4:25:05
and listen to the changes from the post table and event will be St static so
4:25:10
that we can listen to all the events from from this table then we're going to use schema as
4:25:16
public and table name will be posts and the third parameter will be a
4:25:23
call back function for the payload so let's use handle post event which we'll
4:25:29
create later then we're going to subscribe to this channel like this so um here we have an
4:25:38
error um this should have a column like this so we also need to unsubscribe from
4:25:45
these channels so let's return a call back from this use effect and inside
4:25:50
this we can use superbase do remove Channel function and then we need to pause the
4:25:56
post Channel function in here so now let's create this function
4:26:01
handle post event and this will be an async function which will give us the payload so let's
4:26:09
just console log this payload for now and let's see the data that we're
4:26:19
getting like this and let's save this so now let's create a new post and see what
4:26:25
happens testing Real Time
4:26:32
Changes we need to open the terminal to see the logs and then hit
4:26:38
post so the post is created and we got the post event with this this payload and this includes the event type as
4:26:45
insert and this new property contains all the data from our new post as the
4:26:51
body user ID and the file but one thing you will notice that this does not
4:26:56
include the user object and we need that to show in the post card so what we will
4:27:01
do is we will fetch the user inside this function and then update the post State
4:27:06
because we have this user ID so let's just remove this and here we need to make condition if the payload do event
4:27:15
type equals insert that will mean we got a new post and we will also make a
4:27:21
condition if we got the ID in the new object on this payload that will mean we
4:27:26
also got the data for this new post here you can see then uh we will get this new
4:27:32
post from this new object so this is our new post now we also need to fetch the
4:27:38
user for this post so let's use get user data and uh this receives the user ID
4:27:45
here and this will get us the user now let's pause the user ID from this new
4:27:54
post like this so now here we need to set the new post. user uh using this
4:28:01
response so we will check if the response. success is true then we will get the user on the data property
4:28:08
otherwise we will just set it to an empty object so now we have the new post
4:28:13
and we want to use the set post function this will give us a call back with all the previous posts and we will return an
4:28:20
array but first we will add the new post then we will spread all the previous posts so this way our new post will show
4:28:27
on the top so let's save this and see if this is working so let's reload the
4:28:33
application and then create a new post um let me just log out and use a
4:28:39
different account because with this account I've add did so many posts so let's use Jon Snow's account and then we
4:28:46
will create a new post from this account let's log
4:28:54
in not now so now let's create a new post testing real
4:29:03
time then pit post so as you can see this updated the
4:29:08
post State because of the real time changes so let's create a new post and
4:29:14
this time let's use an image uh let's choose this first
4:29:19
image and then hit
4:29:24
post okay so as the post is created it instantly appears on the feed because of
4:29:30
the realtime changes that we implemented now let's open the Android emulator and
4:29:36
see real time changes on both the phones side by side
4:29:44
uh let's just close this and move the iPhone simulator on the same window as well
4:29:52
so here and uh we'll see the real time updates here so let's press a to open on
4:29:58
the Android and uh let's press R to reload
4:30:03
the applications so there we can see the app is running on the Android emulator now
4:30:10
let's log in using my account also this is the first time we using
4:30:15
this application on the Android emulator so let's see how everything goes so we
4:30:21
can see the home screen and all the posts looking good now let's create a new post and see what happens testing to
4:30:29
and let's include an image uh let's use this and then hit
4:30:36
post so as soon as the post is created it appears on both of the devices and
4:30:42
this is because of the real-time changes that we implemented on the post table so
4:30:47
the UI on the Android is looking good we can go to the profile screen we can edit the profile and if I choose the image uh
4:30:55
we don't have any image on this emulator so let's go back the Alert model is
4:31:00
working now let's see if the video player is working on Android so I can play the
4:31:06
video then I can pause the video so the controls are working and the UI is
4:31:11
looking looking good on the Android as well so this is how we can implement the
4:31:17
real time changes from super base so now I want to work on pagination
Likes, Shares & Pagination
4:31:23
and the idea is if we scroll down it will fetch more post but uh we will need to create some test PST so that we have
4:31:30
enough post to test the pagination so let's create these test posts
4:31:48
okay so let's not create all the post from this account let's create another account and then add some post from this
4:31:55
new account so let me just create a new
4:32:10
account okay so the account is created now let me just quickly update his profile data and add a phone number a
4:32:18
location and a
4:32:25
bio and then hit update uh let's also add a profile
4:32:32
picture um let's choose this one and then hit
4:32:38
update okay so the profile is updated now let's create a new new post from uh this new
4:32:44
account let's choose an image uh let's choose this
4:32:55
one and then hit post so there we can see the new post
4:33:02
and I think uh we have enough post to test the pagination so uh here we have a
4:33:07
limit variable which is by default zero and this increases by uh value 10 every
4:33:13
time we call this function and we are posing this to fetch post function and we are limiting the amount of post that
4:33:19
we getting so now we want to create a function that will trigger every time we hit the bottom of this list and we can
4:33:26
use a property on and reached and this will give us a call back and this will trigger every time we hit the bottom of
4:33:33
this flat list so let's just console log a message got to the end and let's save
4:33:39
this and let's just open the terminal and see what happens if we go to the bottom so let me just
4:33:46
refresh and now if I scroll down we should see a message in the console so there we can see got to the
4:33:54
end and it will happen every time we scroll to the bottom we can also add a threshold in pixels so if I add zero
4:34:01
that will mean uh it will trigger the function exactly when we hit the bottom so now in this call back I want to use
4:34:08
the get post function and this will trigger every time we hit the bottom of this flat list and inside this function
4:34:15
we are basically increasing the limit and then passing it to fetch post uh let's just reduce the limit to
4:34:21
four because we don't have a lot of post so let's see how this works so by
4:34:27
default we are seeing a four latest post and as I scroll down uh the get post
4:34:32
function gets called and now we have more post from the database um actually we shouldn't see
4:34:40
this many posts uh it should have fetched only the eight or 12 posts so
4:34:46
for some reason I think it's uh calling this function multiple times so let me just refresh the
4:34:54
app and uh now we should see only the four post and as I scroll
4:35:00
down it should update the limit to 8 but it's updating to 12 so I think I know
4:35:06
why this is happening so if I show you the initial logs it should have called this function two
4:35:12
times there we can see two console logs so basically what happened is when we
4:35:18
have a an empty flat list this on and reach function gets triggered so now we no longer need to call this function
4:35:25
inside the use effect so now let's comment this out and see if this is
4:35:33
working okay so uh let's scroll to the bottom and by default we are seeing four
4:35:38
latest post and now if I scroll down it will call this function and increase the
4:35:43
limit by eight so if I scroll down there we can see fetching eight
4:35:49
Post in the console and as I scroll down it will increase the limit by four so
4:35:55
now we can see fetching 12 post so our pagination is working but there is still
4:36:00
some issues so as I scroll down and fetch all the post from the database it
4:36:05
will still call this function and Trigger every time we hit the bottom as you can see so we need to stop calling
4:36:12
this API when we have all the post from the database and for that we need to use a state has
4:36:18
more and by default this will be true and until this is true we will keep calling this API so let's use this and I
4:36:27
want to make it false when we get all the post so here in this condition we
4:36:32
want to make another condition as post. length equals uh to the post that we
4:36:38
fetching from the database and if this equals that means there's no new post coming so we can set has Moree property
4:36:45
as false and once this is false we can add a condition here at the Top If has Moree
4:36:53
property is false then we will just return null and we won call this API so
4:36:58
let's just save this and at the bottom I want to show a message when we have
4:37:04
fetched all the post so let's add a condition if has more is true then we'll show The Loading and if this is false
4:37:11
then we will show a message so let's add a view and give this a style of margin
4:37:18
vertical as 30 pixels and inside this let's add a text
4:37:25
component and give this a style of no
4:37:30
posts then it will say no more posts and I save this so if I show you
4:37:37
the styles for this we have a font size align to the center and the color from our theme object so now let's see if
4:37:45
this is working so as I scroll down we see the loading and it has feted more post let's
4:37:53
scroll more and now we see more
4:37:59
posts and when we have feted all the post it will say a message no more post
4:38:04
now if I hit the bottom again and again it won't call the API it will just show a message got to the end so this is
4:38:11
working perfectly fine so now let's see our
4:38:19
document so we have implemented real time data fetching post vagination now I
4:38:24
want to work on like post so uh when we hit this hard icon this will like the
4:38:29
post so let's go into our post card here we have our post card and here if we uh
4:38:36
click on this icon uh this is the hard icon I want to add a function on like so let's
4:38:43
add an on press method here and this will trigger on like function let's create this function
4:38:54
here this will be an async function and we going to call another API call inside
4:38:59
this so let's create another function into our post service let's just copy this function and rename this to create
4:39:08
post like and this will receive an object with the post like
4:39:14
data so let's just remove this code uh let's keep the
4:39:19
error and uh let's change this to post like error and this will say could not
4:39:25
like the post so now I want to insert a row
4:39:32
inside post likes table and we will get a response as data and an error then
4:39:39
let's use our subis client and from the post likes
4:39:45
table we want to insert this object as post like and then we want to also select the
4:39:54
data that we insert into the table and this will give us an array but I want to
4:39:59
use only the single object so let's use a single and this will return the data
4:40:04
that is created so uh if we have an error this will return the error but otherwise it will return the success
4:40:11
respon response now let's just copy this and we're going to use another function remove post
4:40:17
like and this will receive a post ID and the user ID like this now when we remove
4:40:24
the data we will only get an error so let's just remove this data and we want to use the delete function which will
4:40:31
delete the data from this table based on some conditions so to add the conditions
4:40:36
we're going to use an EQ function which will basically check if the user ID
4:40:42
equals to the user ID that we're posting to this function and same for the Post ID like
4:40:48
this so if we don't get any error that means data is removed but if we do get
4:40:53
any error we just return a response as could not remove the post
4:41:00
like now let's use this function inside the post code component and first we need to create the data and this will
4:41:07
include the user ID which we have on the current user so current user. ID then
4:41:13
the post ID will be item. ID now let's call this function that we
4:41:20
created and this will give us a response create post like and pause this
4:41:28
data and once we get the response we'll just console log and see uh if it
4:41:34
successfully created the like and I also want to show the user if the like is not
4:41:40
created so if result. success is false then we want to show an
4:41:47
alert the title will be post and the message will say something went
4:41:56
wrong okay so now let's save this and open the console and see if this is
4:42:03
working so now if I click on this hard icon uh we see a response as uh the
4:42:09
success is true so it must have created the data in the super Bas so let's see
4:42:16
let's go to table editor and go into post likes so there we can see our first
4:42:22
post like we have the post ID here and we have the user
4:42:27
ID so now that we have this data we can include this while fetching the post
4:42:32
data and we want to show the post likes here on the postcard currently we don't see the new like here so let's just
4:42:39
refresh so because we have a foreign key relationship with the post and the post
4:42:45
likes we can include this table in while fetching the post and it will include
4:42:50
the post likes for each post so just like the users table we can specify the
4:42:55
post likes table and the data that we want to fetch and I want to fetch all the data so let's specify static and
4:43:02
let's save this now let's console log the post item and see if you're getting
4:43:07
the post likes array so uh let's cons so log the data
4:43:13
here post item and let's save
4:43:19
this so we don't see the post likes so let's just refresh the app and see if we
4:43:26
get this array um fesh post error unexpected end of the
4:43:35
input um what did we do wrong so let's go into our function
4:43:41
fetch posts everything looks all right so
4:43:49
let's just refresh the app and check again uh still the same
4:43:55
error maybe uh we need to remove this comma at the end so let's save this and
4:44:03
see okay so now we getting the data so it was a comma make sure you don't add
4:44:08
this comma at the end so now we getting the post likes array so let's go into
4:44:14
the post card and let's just remove this console log now I want to set this post
4:44:20
likes array into a likes state so let's do this inside a use effect hook let's
4:44:27
add an empty dependency array and I want to set likes uh let's just create this
4:44:32
state here likes and ass set likes by default this will be an empty array and we will
4:44:40
set the like likes from the post item so uh inside this use effect hook let's use
4:44:46
set likes and the item. post likes and this state is showing red
4:44:52
lines because we have another likes constant so let's just remove this and for this liked Boolean here I want to
4:44:59
filter all the likes and see uh if there is any like by the current user so let's
4:45:05
filter the likes and for each like we will check if the like. user ID equals the current user ID that means this user
4:45:13
liked this post and we will check if we have any first item based on this we
4:45:19
will return true or false like this and let's save this so
4:45:25
as I save this you can see it's showing this post is like and the count is one
4:45:30
and that's because we already used these values on this hard icon so here you can
4:45:35
see the likes. length but there is one more issue so if I like any other
4:45:43
post so in the console you can see the like is created but it's not updated in
4:45:48
the post card and we need to update this likes array so before we call this API
4:45:54
let's set the likes here and first we need to spread all the previous likes and then this new data here so this will
4:46:02
update the likes array and I'll show our latest like so let's just
4:46:08
refresh so now if I try try to like a post it instantly updates the post likes
4:46:16
and we see this on the postcard but if I click on it again it will again create a
4:46:22
post like and update the account but this shouldn't happen it should remove the post like so here we need to make a
4:46:29
condition if user has already liked the post then we will remove the post like
4:46:34
so let's add a condition if liked is true then remove the like otherwise uh
4:46:39
create the new like so let's add this code
4:46:46
here and we're going to use the same code for remove like as well but uh
4:46:51
let's just remove this data and here first we need to update the previous likes so let's filter all the likes and
4:46:58
remove the likes where the user ID uh is the current user ID so this will give us
4:47:04
all the likes where the user ID is not equal to the current user. ID and we
4:47:11
need to update this into the likes state so we no longer need to add this data so
4:47:16
we just need to spread all the updated likes here like this and instead of calling the create post like we need to
4:47:23
call a remove post like and here let's see all the
4:47:31
parameters so first we need to pause the post ID and we have item. ID then for
4:47:37
the user let's use current user. ID and and this will give us a response and if
4:47:42
there is any error we will just show the error so uh for the console log message
4:47:48
this will say removed like and for this one it will say added like like this and
4:47:55
let's save this so let's just refresh the app and see if this is
4:48:00
working and let's click on this post and we got an error something went
4:48:07
wrong could not remove the post like data doesn't exist so yeah we're not
4:48:12
using the data inside this function so let's just remove this property and this will return success true when the data
4:48:19
is removed but it must have removed the data from the database so let's just refresh and check
4:48:26
again and yes the likes are removed now if I remove the like from this post this
4:48:31
will remove the like and if I click on it again this will create the like so I can click on this as many times as I
4:48:38
want and this will work perfectly f M So currently this post is liked and if I
4:48:44
refresh the app we should see our like here so this is working fine so now I want to work on this share
4:48:52
icon so if we click on this it should share the image or the post content so
4:48:57
let's go to this
4:49:04
icon and here we have the share icon so let's add an onpress function and this
4:49:09
will trigger on share share function so let's create this new function
4:49:15
here this will be an async function because later we're going to add an API
4:49:20
to download the files from the post and we're going to use the same library that we used before Expo file system here is
4:49:27
the library but for now uh we want to implement only share the content so
4:49:33
let's create the content and we will have a message as item. bu and then we
4:49:39
can use the share functionality from react native then using the share function we can provide this
4:49:46
content and let's save this now if I click on the share icon uh we see the
4:49:51
share model but I don't want to share the HTML content I only want to share the text so let me just paste a function
4:50:00
here so this function uses Rex to remove the tags from a string so you can just
4:50:06
copy this from here now let's use this on item body
4:50:11
and this will remove all the tags and all the styling from the content and give us only the raw text and that is
4:50:18
what we need so let's save this and now if I share this content we only see the text
4:50:25
we don't see HTML tags now so I can share this one and we only see the
4:50:31
content without the tags now let me find a post with some
4:50:37
styling so here we have a new text as Bard but but if I share this we won't see any style we will see only the text
4:50:45
but if I share a post with media we won't see anything because we're only sharing the item body we're not sharing
4:50:52
the item file so uh first we will have to make a check if we have a file on this post then uh we will download the
4:50:59
file and then share the URI for that file we can only share the local file
4:51:05
URI because I've tried with the remote files and it doesn't work so let's let's
4:51:10
go into our image service and here we need to create a function download
4:51:16
file this will also be an Asing function and it will receive the URL for the
4:51:22
remote file so let's add a TR catch block and if we get any error while
4:51:29
downloading the file we'll just return null so now we going to use the file
4:51:35
system library and we're going to download the file and it will give us a response with a URI and uh the function
4:51:43
that we're going to call is download async and it requires the URI for the
4:51:49
remote file and then the local file path where we want to store this file so for
4:51:54
this one let's just create a function get local file path and we will pass the URL
4:52:02
here so now let's create this function and this will receive the file
4:52:10
path now we also need to get the file name from this path so let's create a variable and the way we can get the file
4:52:17
name is we need to split uh this URI uh by slash and this will uh return an
4:52:24
array with all the poths and the final path will be the file name so we can use the Bob function and uh then we will
4:52:31
have the file name so now we need to return the full uh URI so we can use the
4:52:37
file system do document do directory and then the file name so this is where the
4:52:45
file will be downloaded and then we can use this to share the file so let's use
4:52:51
this function here and then return the URI like this now let's use this
4:52:57
function here to download the file so if we have item. file then we will get the
4:53:02
URI using this download file function and here we need to use the
4:53:07
function get superbase file URL because we need to pause the full remote URL of
4:53:12
this file and this function basically returns an object with the URI and we don't need
4:53:19
the whole object we just need the URI property so uh let's just use U on this
4:53:26
function now this will uh get us the full remote path and then download the file now we need to set the content. UR
4:53:34
property as this local URI from our system then if we share this cont
4:53:40
content this will hopefully show this media so let's just click on share
4:53:45
hopefully it's downloading the file and we don't see
4:53:51
anything um did we do something wrong oh okay so this needs to be URL so
4:53:58
let's just change this to URL as well and this one
4:54:04
too okay so let's save this and now let's try again okay so now we can see it's
4:54:11
actually sharing the image so we can copy this save this or if we have other apps we can share it to them so now I
4:54:19
want to show a loading State on this icon because when we share the media it takes some time to download the files so
4:54:26
let's add a loading State and by default this will be false and we need to make it true when
4:54:34
we download the file inside this condition and then make it false after
4:54:41
we get the local file URL so using this inside our icon here
4:54:48
uh we need to make a condition if the loading is true then we need to show the loading
4:54:54
component so let's import the loading component and size will be
4:54:59
small let's close this and if the loading is false we can just show this share
4:55:06
icon like this and let's save this and see if this works so now if I share this
4:55:12
image we see the loading until the file is downloaded and then we see the share
4:55:18
model to share this media so uh let's test this on this file if I click on
4:55:23
share we see the loading the file is downloading and then we can share this
4:55:29
file so with this we are able to share the text content and the media files
4:55:35
then we can like or dislike any post next we're going to work on creating and
4:55:40
showing these commments but before that let's go into our
4:55:45
document and next we are going to work on our post details model so here are
4:55:52
all the steps so let's just create a mockup for this
4:55:57
model this will be our post details model
4:56:03
screen and uh first this will show all the post details and we're going to use the post card that we used in the home
4:56:10
screen for showing the post details so let's just copy this and let's just use it here on the
4:56:18
top so first it will show the post details and then at the bottom we can
4:56:23
add uh the comment list so uh before the comment list we want to add an input box
4:56:29
here and this is where user can add the new comment and we will have this icon
4:56:36
here which if user press on this this will create a new comment and update the comment list then after this we will
4:56:44
have a list of the comments and uh let's just create a code for each list item
4:56:50
and on the top we will have the user info who created this comment and we will have the user Avatar and then the
4:56:58
username so let's just copy this from here then after this we want to show the
4:57:05
comment body so let's add the comment body and then place after the user
4:57:12
info so this will be for each comment item uh now let's add it here after the
4:57:20
create comment UI and let's add two more
4:57:37
items like this so so now you get the idea of how the post details model will
4:57:44
look like this will be a model not a full screen later we will use this model to
4:57:51
delete or update in the post as well so uh we can open this model in two ways uh
4:57:57
let's go let's go to our post card here so if we click on the comment icon or
4:58:04
this three dots icon this will open the post details model so now let's create
4:58:10
create this new model into our main folder post
4:58:15
details and let's create a functional component post details and let's save
4:58:21
this so now we need to open this model when we click on the comment icon so
4:58:27
let's go into the post card um this is the icons folder uh let's search for the
4:58:34
post card here and now if we click on the comment icon let's add an onpress
4:58:40
function here and this will trigger open post details function and we also need to do this for
4:58:47
the three dots icon so uh we already did that so let's go into this function and
4:58:54
here we need to use the router to move to this new screen and here in the props
4:59:00
you can see we already receiving the router from the parent component so let's use router. push and here we need
4:59:08
to define a path name as this new component that we created so post
4:59:14
details then we need to pass some parameters as the post ID will be item.
4:59:21
ID and we can use this post ID on this next screen to get the post details so
4:59:28
now if we click on this icon uh we move to this new screen but you can see this
4:59:33
is not a model this is a full screen so let's go into the layout file and here
4:59:39
we need to define a new screen so let's use stack. screen
4:59:46
component then here we need to name it as post details and then provide some
4:59:53
options here we need to specify the property presentation as
4:59:59
model so it will open as a model now let's close it and save this so no route
5:00:06
named as post details and this is actually true because this is inside the
5:00:12
main group so we need to specify the main group as well in the name of this screen so now we don't see the warning
5:00:20
and if you click on this now this opens as a model not a full screen so now we need to get the post ID
5:00:27
inside this model that we post from the previous component and we can use a hook called use local search params this will
5:00:35
give us an object with all the parameters and we can EXT the post ID
5:00:41
and console log this ID here so got the post
5:00:46
ID let's save this and open our terminal to see it in the
5:00:52
console so now if I open this model uh we see a message got the post ID as
5:00:59
12 also if you notice these IDs are in an order so for this one it will be 12
5:01:05
and for the next one it will be 11 and that's because superbase uses an integer
5:01:10
for the primary key and then increases it every time we create a new record and
5:01:15
if you have used MySQL or postest before then you already know about
5:01:20
this so now I want to fetch the post details so let's create a
Post Details
5:01:26
state post and set post by default this will be
5:01:32
null and we're going to call a function to fetch the post details inside our use effect hook let's add an empty
5:01:39
dependency array here we need to call a function get post
5:01:45
details and let's create this function here this will be an async
5:01:50
function and for now let's just add a comment here get post details
5:01:57
here so now we need to create an API to fetch the post details from superbas so
5:02:02
let's go into our post service and here let's just copy this
5:02:08
function and change the name to fetch post
5:02:14
details and we will receive a parameter as post ID then we don't need this order
5:02:22
function so let's just remove this and here we will add a function EQ because we're going to get the post based on
5:02:28
this ID so if ID equals this post ID then we'll fetch the data and uh uh we
5:02:35
don't need this limit as well so let's just remove it and we're going to use a single function because we want to get
5:02:42
the single object then let's change the post details error and this message will
5:02:48
say could not fetch the post like this so everything looks all right here now
5:02:54
let's use this inside our post details model and this will give us a response
5:03:00
and we need to pause the post ID like this and we'll just console log
5:03:07
the result got post Det
5:03:12
details so let's save this and see if this is working so let's see in the console if I open the post details here
5:03:19
we can see got post details we have the data and success as true so let's just remove this and here
5:03:27
we'll make a condition if result. success is true then we'll set the post as result. data like this and now I'm
5:03:35
just going to copy and paste the styles for this whole component so I don't have to write them again and I'll just show
5:03:42
you these Styles whenever I use them so let's import WP HP and the theme
5:03:49
object like this and uh let's just give this container a style of
5:03:57
container and inside this uh let's just remove this and here we need to add a scroll
5:04:04
View and let's just hide on the vertical scroll indicator let's make it f
5:04:11
and for the content container style let's use the style of list and uh so for the list style we
5:04:20
have the following style we only have a padding horizontal then inside this let's add our postcard component for the
5:04:26
Post details and here first we need to add the item property which will be the
5:04:32
post and then uh we're going to add the current user so this will be user and we need to
5:04:38
get the user from our use Au hook like this then uh we also need to
5:04:47
define the router because we need the router inside the post card component so
5:04:53
let's add the router here then we can also pause a property
5:04:59
has Shadow and I want to make it false because I don't want a shadow for this component here so let's close it and
5:05:06
save this so we get this error and and this is because initially the post state
5:05:12
was null and we are using this inside this component so we need to make a condition here actually let's just
5:05:19
create a new state and call it start loading and set start loading so
5:05:27
initially this will be true and we'll make it false when we get the post details from super base so right here
5:05:35
we'll just set this to false and we're going to use this is inside this condition so whenever this
5:05:41
is true we're going to return a loading state so let's return a view and give
5:05:47
this a style of Center and for this item I'm using the
5:05:53
following Styles so you can just copy them now let's import our loading
5:05:58
component in here and let's save this so now if I open the model we
5:06:05
should see a loading State and then we can see the post details so this is very
5:06:10
useful because we're using the same component on the home screen and for the Post details model so we don't have to
5:06:18
create extra UI for the Post details but we do want to make some changes inside
5:06:23
this component because right now if I click on this comment icon again this will again open the post details model
5:06:30
and I don't want this to happen so we need to make a condition whenever we use this component inside the post details
5:06:37
model then we will disable these options and also hide the three dotts icon so
5:06:43
for that we will need to pause a new property to post card and let's call it show more icon and we'll make it false
5:06:50
for this component so let's use this inside the post card let's receive it
5:06:56
here and by default this will be true so on the home screen we will still see these icons and actions so let's go to
5:07:05
the three dots icon and we'll make a condition if show more icon is true only
5:07:11
then we will just render this icon like this and let's save this now
5:07:18
we can still see this icon in here but if we open the post details we don't see this icon in the
5:07:25
post details component now let's fix this comment icon so when we click on
5:07:31
this we need to use the same property in here and we need to make a
5:07:36
check if uh show more icon is false then we will just return null like this
5:07:43
so let's save this and see if this is working so now if you click on this it opens the post details and uh if we
5:07:51
click on the comment icon here nothing should happen so this is working so now I want to work on the
5:07:57
create comment UI so let's go into the post details and here let's add a comment comment
5:08:04
input and here we need to create a container so let's create a view and give this a style of input
5:08:12
container and inside this we need to import our input component and let's
5:08:17
give it a placeholder as type
5:08:23
command and for the color of this placeholder let's use the property placeholder text color and we will use
5:08:31
the text light color from our theme like this now uh we want to add the container
5:08:38
Style and we will use flex one and the height of HP
5:08:47
6.2 and about the radius of excel from our
5:08:54
theme like this so let's close it and let's save this so there we can see the
5:09:00
comment box now for the input container we have the following Styles so you can
5:09:06
just copy them now I also want to add a ref reference for this input so let's
5:09:11
create a reference let's just remove this and create an input reference and
5:09:16
use the hug use ref and null so we want to use this because
5:09:24
later I want to clear the input when uh the command is created so we can just
5:09:29
pause it like this and we're using this input reference property here as a
5:09:34
reference for this text input and we also need to create one more reference
5:09:40
for holding the value of this text so let's create a comment reference and by
5:09:46
default this will be empty string and let's add a function on this on change
5:09:53
text and whenever a value gets updated in the input uh this will update the
5:09:59
comment reference. grun value like this so let's save this and now if I
5:10:07
type anything here I should update the reference value and we can use that to
5:10:13
create a comment now we're going to add a button here so let's use touchable
5:10:21
opacity and let's give this a style of send
5:10:27
icon let's close this and we're going to use the send icon here so name will be
5:10:33
send and for the color we're going to use primary dark color from our theme
5:10:41
let's close it and let's save this so there we can see the button now
5:10:46
when we click on this button it should trigger a function on new command so
5:10:51
let's add this function here and let's make it capital so let's
5:10:57
uh Define this function here and make it a
5:11:03
sync so when we create the command we first need to start the loading so let's
5:11:08
create a loading State set loading and by default this will be
5:11:15
false now let's use this where we use the icon so when the loading is true we
5:11:20
will show a loading State otherwise we'll just show this
5:11:26
icon so let's copy it and move it here and for the loading let's create a
5:11:32
view and give this a style of loading and inside this we can use use
5:11:39
our loading component with a size of
5:11:47
small like this let's close it and save this so we can't see this loading but if
5:11:54
we make this loading State true like this then we can see uh this
5:12:01
is how the loading state will show so let's make it false again okay so now I want to create the
5:12:08
data for this new comment but first let's add a condition here we need to check if we don't have the current value
5:12:16
inside the comment reference then we can return null but if we do have a value
5:12:21
then we will create a data object and inside this we will specify the user ID
5:12:27
from the user object and the post ID from the post object and for the comment uh we need to
5:12:34
specify the text property as the comment reference Dot current
5:12:41
value so this will be the data for the new command and now we need to work on
5:12:47
uh saving this command into the super base so for that we will need to go into
5:12:52
the post service and here let's just copy this function and create post
5:12:59
like and paste this here change the name to create comment and this will receive
5:13:06
the comment data now let's just change this error to
5:13:11
comment error and the message will say could not create your
5:13:17
comment like this so this will save data into the comments table and we need to
5:13:23
insert the comment data like this and we can leave everything else as it is so let's go
5:13:31
into our post details component and here first we need to start the loading then
5:13:36
let's use this function and this will give us a response and we need to pass the
5:13:43
data and after this we can set the loading to
5:13:48
false then here we need to check if the result. success is true that means the
5:13:53
command is created but if it's not then we need to show an alert to user so
5:13:59
title will be comment and for the message we use result. message property like this so here we need to clear the
5:14:07
references so input reference do current and we can use the clear function that will clear this input but this will not
5:14:15
clear the comment value so for that we need to pause an empty string for the
5:14:20
comment reference and later we will send a notification to the post owner whenever a comment is created so let's
5:14:27
just add a comment for now um optional chaining assign isn't
5:14:33
currently enabled uh so we can't use this question mark here when when assigning a value
5:14:41
okay so that will fix the issue so let's just refresh the app and see if we can
5:14:46
create a new commment let's open the post details and here let's just test it and
5:14:53
add a new comment as cute and let's send it so now let's see in the console
5:15:01
actually we did not console logged any response so we need to go into the super base
5:15:09
and then open the comments table so there we can see our new comment as cued
5:15:15
and this includes the post ID and the user ID so this is great now next we're
5:15:21
going to work on a list of all the comments and we're going to show it at the bottom here but before that let me
5:15:28
just create another command and we will work on showing the comments count in the post card component so let's add
5:15:36
this comment and hit send so the comment is hopefully created now
5:15:42
let's add the comments count in the post card and for that we will need to change the API for fetching the posts so let's
5:15:50
go into the post service and we can do that because we have a relationship with
5:15:55
the post and the comments so we can include all the comments within a post
5:16:01
so uh here is the API and here we need to add the reference table as comments
5:16:08
then we can PA a property as count which is not added in the comments table but when we use this this will count all the
5:16:15
comments for this specific post and we can check this inside the post card
5:16:20
component so here let's just console log the post
5:16:29
item actually let's just console log the comments property directly because we
5:16:34
just need to see that let's save this and we can see it's
5:16:40
undefined but let's just uh refresh the app so now we can see it has an array
5:16:47
with an object and this includes a property count which is the number of
5:16:52
comments on the post so now we can use this so let's just remove
5:16:58
this and let's go to the comment
5:17:04
section and here so here we need to chose item.
5:17:11
comments and we need to go to the first object because this is an array and we
5:17:16
need to show the count property so let's save this so there we can see the count
5:17:22
and we made two comments earlier so this is correct now if I open the post
5:17:27
details we will hopefully get an error like this so this is because the post
5:17:34
details API does not include the comments table we do have a relationship with the comments and the post but we
5:17:41
just haven't added the comments table yet and because we're using the post card component and this expects the
5:17:48
comments array so this is why we're getting this error so now let's add the
5:17:53
comments table inside this API and for this one I want to fetch all the
5:17:58
comments so I will use atic then I also want to fetch the user who created this
5:18:04
command so I can use the users table and specify the properties that I want to
5:18:09
use so we can do this because we have two relationships inside the comments
5:18:15
table so if I edit the comment table we can see two relationships with the post
5:18:22
and the user so we can do this and this will get us the user inside the user's
5:18:28
property but we can change this to user like this because we will only have one user for each comment so now we need to
5:18:36
fetch all the comments in a descending order and for that we can use a function order
5:18:42
where I can specify the column as created ad and some options where
5:18:48
ascending will be false and we have a property as foreign table which will be
5:18:54
the included table as comments now this will order all the comments as descending and we will get
5:19:02
all the latest comments on the top so let's just save this and open the post
5:19:09
details so now we not getting this error but we also don't see the comments count
5:19:14
and that's because now we getting the comments array not the count property so
5:19:19
if I go into the post details here let's just console log the post
5:19:28
object like this and let's save this so let's go to the bottom and here we can
5:19:35
see it has the body ID there we see the post
5:19:40
likes and there we can see the comments array and this includes all the comments not
5:19:47
the count property so we need to override this because we are using this inside the post card component so let's
5:19:55
spread the post data here then for the comments property let's choose array and
5:20:01
then an object with the count property and we need to pause the length of all
5:20:07
the comments on this post object like this and let's save this so
5:20:13
now we can see the count as true which is correct we made two comments earlier
5:20:19
also let's just add these question marks so if we don't get these values this won't throw an error so for this post we
5:20:27
have zero comments and if I open the post details this shows correctly so everything's working
5:20:33
great now for some reason if we're not able to fetch the post details or the post is deleted then I want to show a
5:20:40
message here so let's add a condition here if we don't have a post in that
5:20:46
case we will just return a view and let's give it a style of
5:20:54
Center and I want to show it on the top so for that let's just add a style of
5:21:00
justify content as Flex start and then I want to have a space on
5:21:07
the top so let's add a margin top of 100 pixels then inside this let's use a text
5:21:14
component and give this a style of not found and this will just say post not
5:21:24
found so let's save this and if we don't have a post now it will show this
5:21:30
message so let's just fetch a post with a random ID and let's save this so now
5:21:36
if I open the post details model this will say post not found because we
5:21:42
don't have a post with the ID 234 so this is correct and now let's change
5:21:48
this back to post ID and now if I open the post details this will show the post details like
5:21:55
this so now let's just minimize this and work on our comment list so here let's
5:22:03
just add a comment comment list
5:22:08
then let's add a view and give this a style of margin vertical of 15
5:22:16
pixel and then I also want to have a gap of 17 pixels between the
5:22:21
items and here we need to map through all the comments on this post so let's
5:22:27
use the map function and for each comment I want to return a comment item
5:22:33
component which we haven't created yet so we'll create this in a second and we need to pause the item as the commment
5:22:40
so let's close this and now we need to create this uh component inside this
5:22:49
folder then let's create a functional component let's save this and now let's
5:22:56
just import this here and let's save this so there we can
5:23:02
see two commments items and we need to add the key property on this component
5:23:08
and this will be the ID so let's convert it to the string like this so now if we
5:23:15
don't have any comments I want to show a message so let's make a condition here if comments. length equals z that means
5:23:23
we don't have any comments and in that case I want to add a text component here
5:23:29
and let's give it a style I want to use the color of text from our
5:23:34
theme and then let's have a margin left of five 5
5:23:40
pixels and in here I want to say a message be first to comment on this post
5:23:46
like this now let's go into the comment item and here we need to receive the
5:23:53
item as the comment and uh now let's give this container a style of
5:24:00
container and for the Styles let me just copy and paste the stylesheet for this
5:24:05
component and I'll show you the Styles whenever I use so here is a
5:24:11
stylesheet and now let's just import the theme object and we also need to import this
5:24:18
HP function as well so now let's give this a style of container and you can see the styles for
5:24:25
the container in the Styles sheet now let's import the Avatar component and give this a URI of the user image so
5:24:33
user can be found in the item because we included the user in each command item so let's close this and let's save this
5:24:40
so there we can see the default image for the user now after this I want to add another
5:24:47
view and let's give this a style of
5:24:52
content and inside this let's add another view and let's give it a flex direction
5:25:00
of row justify content as space between and align items Center so the
5:25:07
idea is I want to show the user name and the creation date on one side and on the other side I want to show a delete icon
5:25:15
so that's why we using Flex direction as row now let's create another uh View and
5:25:21
give this a style of name container like this so these are the styles for the
5:25:26
name container and the text so let's use a text component and give this a style
5:25:33
of text and here I want to show the username so this can be found on item.
5:25:41
user.name let's save this so there we can see the
5:25:46
username also this background color is coming from this content style so you can just copy the content styles from
5:25:54
here so next we want to add the creation date so let's just copy this component
5:25:59
and uh let's use the created ad property from the item so item do created
5:26:06
at let's save this so I know this looks ugly but we'll fix this in a second but
5:26:12
let's just change the color to text light color for this
5:26:23
date okay so now we want to format this date so let's create a new created ad
5:26:29
property and we're going to use moment library to format this date so let's
5:26:34
provide item. created ad and then the format function with this
5:26:40
format so let's use this created ad property here let's save this now it looks good
5:26:47
now I want to add a middle dot between the name and the date so let's use text and then add this dot so now it looks
5:26:55
good so next I want to add a delete icon on the right side so let's add a
5:27:01
touchable opacity and inside this we'll use our
5:27:06
icon component and give this is a name of delete then the size will be 20 and for
5:27:14
the color we're going to use the rose color from our theme so let's close this and save this
5:27:21
and we will see the icon on the right because of FX direction as row and justify space between so now I want to
5:27:29
add a condition where this will only show if this property is true so we're
5:27:34
going to use scan and delete property and if this is true only then we will show this delete icon and we'll pass
5:27:41
this property from the parent component so let's just receive it here and by default this will be false like this so
5:27:48
if I make it true we can see the delete icon so by default let's just make it false so now we're going to show the
5:27:55
comment text so let's add a text component and give this a style of text
5:28:00
and I want to have the font fate as normal for this text and inside this let's use it item.
5:28:08
text property and let's save this so there we can see our two comments that
5:28:14
we made earlier and I just noticed that it's not displaying the user image properly
5:28:21
because this is the same account and uh we can see the user has an image but on
5:28:27
the comments it does not so where did we use avar component so
5:28:35
yeah here we need to specify the user there was a spelling mistake so now we
5:28:40
can see the user image now let's just log out and log in with a different account because this user is making
5:28:47
comments on his own post so let's just log in with my
5:29:00
account okay so if we go into the post details here we don't see the delete icon and that's because by default this
5:29:08
scan delete property is false and this will be true based on a condition from the parent component so here let's pause
5:29:15
this property so this will be true when the current user ID equals to the
5:29:21
comment. user ID means this user created this command or if the current user is
5:29:26
the owner of this post so user. ID equals to the post. user ID in both
5:29:32
cases we will show this icon so we don't see this icon now because uh we did not
5:29:38
created these comments or this post but if we create a new comment on this post
5:29:43
this will show the delete icon for us so let's create a test command so the comment is created but it does not show
5:29:50
automatically which we will handle later so for now let's just refresh the app and check the latest
5:29:56
comment and there we can see the delete icon because we created this comment and
5:30:02
we can delete this command now when we click on this this should show an alert
5:30:07
to the user if if he wants to delete this comment or not so for that let's create a function inside the comment
5:30:13
item and let's call it handle delete so this will use an
5:30:20
alert and using the alert we want to show the title as confirm and for the options uh let's just use the same alert
5:30:28
that we used in the profile for logging out so let's just copy this and paste
5:30:34
this here so title will be confirmed and for the message Mage I want to say are you sure you want to do
5:30:41
this and let's just leave the cancel as it is and for the log out it will say
5:30:47
delete and when we click on this this will trigger an on delete function where I want to pass the comment item as well
5:30:55
and this will be received from the parent component so let's receive it here and by default this will be an
5:31:01
empty function like this so let's save this and now we can use this handle
5:31:07
delete function on this touchable opacity so when we click on this this will show an alert so let's save this
5:31:15
and now if we click on this we see the alert so now we need to implement this
5:31:20
on delete function in the parent component so let's pause the on delete function to this
5:31:29
component and now let's create this on delete comment function
5:31:35
here this will be an async function which will receive the item as the
5:31:41
comment and for now let's just console log the comment that we are
5:31:48
deleting let's save this and see if this is working so just open the terminal and
5:31:53
if we click on delete here we see deleting this command so now we need to
5:31:59
work on the API that will delete this comment from the super base so let's go into our post service
5:32:10
and here let's just copy this remove post like function and paste this here and let's
5:32:17
change the name to remove comment and this will receive the comment
5:32:26
ID and we're going to remove a row from the comments table then uh we need only
5:32:32
one condition so let's just remove this function and this will check if the ID
5:32:37
is is equal to the comment ID like this so let's change this to remove comment
5:32:45
error and the message will say could not remove the
5:32:52
comment like this and once the comment is deleted this will give us a success
5:32:57
as true but we also want the comment ID back so let's return the data as comment
5:33:03
ID so let's save this and now we can use this here so let's get the result from this API
5:33:12
remove comment and we need to pause the comment ID so comment.
5:33:18
ID and once this API is completed we will check if the result. success is
5:33:23
true that means the comment is created deleted and if it's not then we will
5:33:29
alert the user with a message from our response so result. message like this so
5:33:36
if the comment is deled we need to update the post so let's use set post and this will give us the previous post
5:33:43
in the call back and we need to update this so let's create this updated post
5:33:48
here let's spread all the previous post data and then we need to update the post
5:33:55
comments list and let's just filter out the comment that is deleted so let's use the filter function
5:34:03
on this and we need to keep all the comments where the ID is not equal to
5:34:08
the comment ID that was deleted like this and then we need to return this
5:34:14
updated post and this will hopefully update the comment list as well now
5:34:19
let's see if this is working so let me just refresh the
5:34:27
app and let's open the post details and let's just delete this first
5:34:33
test comment and hit delete so there we can see the comment is gone
5:34:40
and it also updated the comments count because this was also coming from the post data so in the console you can see
5:34:47
deleting this comment and this comment is gone now let me just refresh the app
5:34:53
and see if the comment is actually gone so there you can see the count is
5:34:58
two and the comment is gone from the database so uh there is one more issue
5:35:05
with the comment list so now if I create new comment and hit send uh you'll see the comment is
5:35:12
created but it does not update the comment list and for that I'm going to use the real time updates from subab
5:35:18
base which we used for the new post so let's go into the home and let's just
5:35:23
copy the same logic we're going to make some changes to this but most of this will be the same so let's just copy this
5:35:31
and move this to post details here we already have a US effect
5:35:36
hook so let's just copy this get post details function and paste it here now
5:35:42
we don't need this use effect hook okay so now uh let's change the name to
5:35:48
comment Channel and the channel will say
5:35:54
comments and for the events we only want to listen for the insert events whenever
5:35:59
a new comment is created then schema will be public and for the table let's
5:36:04
use comments table and let me just format this so this will
5:36:10
listen to the inserts for all the comments but I only want to listen for the comments on this specific post ID
5:36:17
and for that we can use this filtered property so let me just show you the documentation for
5:36:24
this um listen to updates
5:36:30
no uh listen to R level changes yes this is what we need so we
5:36:36
can basically fill filter the events for a specific post ID using this method so
5:36:42
we're going to use this filter with the post ID from this post details model if we don't do this this will fetch all the
5:36:49
inserts for all the comments so let's use the post ID equals to the EQ
5:36:56
function and then the post ID value like this uh let's see the syntax
5:37:03
um we need this dot as well so let's use a DOT after the EQ function so this will
5:37:10
receive all the comments for this specific post so now let's import the superbas client and let's save
5:37:17
this um let's create a new function as handle new comment and this will receive the new
5:37:24
comment payload so let's create this function here this will also be an Asing function
5:37:30
because we are going to use the user data for each comment as well so we need to make it async to fetch the user data
5:37:37
data later so let's open the comment details okay so inside this let's just
5:37:44
console log the payload first got new comment and we get the new comment in
5:37:52
the new property so let's just console log this and here let's make a condition if we have new data inside this payload
5:37:59
then it means a new comment is created and for that let's create a new comment
5:38:04
property and spread all the data from a new property now let's fetch the user for
5:38:10
this comment so this will give us a response and we're going to use the get user data function and let's pause the
5:38:18
user ID from this new comment and we'll set the user property
5:38:25
of the new comment if the success is true on this response then let's use result. data
5:38:33
otherwise this will be an empty object like this so now we need to update this
5:38:38
comment and for that we need to call this set post function this will give us the previous post and we can just spread
5:38:46
the data from the previous post but we need to update the comments so we need to show the comments on this new comment
5:38:53
on the top so let's use an array but first object will be the new comment and then we'll just spread all the previous
5:38:59
comments like this so now whenever we create a new comment this will update
5:39:04
the post and the comment list and we will see it in the terminal as well now let's create a new comment as test two
5:39:12
and hit send so there we can see it in the
5:39:18
terminal got a new comment and this also updated the post and the comment list so
5:39:23
this is working fine so now let's just delete this and the comment is deleted let's
5:39:29
delete this as well okay so the delete is working and the new comment is working with the
5:39:36
realtime updates so let's go into our document and for the Post details model we have completed
5:39:43
all of these steps but we will uh work on the remove and the edit post next and
5:39:49
these icons will be added to this post details model as well but before that I
5:39:54
want to show you guys one more issue with the comments count and you guys will fix this
5:40:01
so on the home screen we see the comments count in each postcard so if I
5:40:06
open the post Det details and then create a new comment let's create a test comment and then hit send so we see the
5:40:14
count is updated and we see the new comment but if I go to previous screen we don't see this count updated so to
5:40:20
fix this you will need to listen for the comment inserts inside the home screen in this use effect and whenever a new
5:40:27
comment is created you will need to Loop through all the post and then update the comment count and this is very easy as
5:40:34
I've just shown you guys how to implement this so I will leave this up to you guys you can play with this and
5:40:40
add this feature so now if we open the post details model here I want to show some
Update Post & User Profile
5:40:46
icons on the top to delete or edit the post so for that first we will pass a
5:40:52
property into the post card to show the delete icon this will be true for this
5:40:58
component and then we're going to pause some functions to delete or edit the post so we will create a function on
5:41:05
delete post and and then on edit we will trigger on edit post function now let's
5:41:12
create these functions inside this component this will be asnc function
5:41:18
which will receive the item as the post let's just console log the item as
5:41:24
delete post like this and let's just copy this for the edit post as
5:41:32
well and this will say edit post okay so now let's save this and go
5:41:38
into the postcard component and receive these properties by default a show delete property will be false because I
5:41:45
don't want to show these icons on the home screen and uh for functions uh let's use empty functions by
5:41:53
default we will only show these actions whenever the short lead property is true
5:41:58
inside the post details model and we're going to add one more condition as if the current user is the post owner only
5:42:06
then we will show these icons so let's add a condition here if show delete is true and if the current user ID equals
5:42:14
to the item. user ID that means this user created this post so he can delete
5:42:20
or edit this post so in that case we will just return a view and give this a style of
5:42:27
icons and let's close this um I don't think we have the icons
5:42:34
style I think it's the actions so let's say these two actions and we have the
5:42:40
following styles for the actions we're using Flex direction as row because we're going to show two icons in a row
5:42:46
so let's just copy in this action and paste it
5:42:52
here so this will be edit icon and when we click on this this will trigger on edit function and let's change the icon
5:42:59
name to edit then let's change the size to
5:43:05
2.5 and let's save this so we won't see this on this post details because this is not my post so
5:43:12
let's see if we have any post from this
5:43:18
account um here we have my post so if I open the post details there we can see
5:43:24
the edit icon but let's just remove the stroke
5:43:29
width and let's save this so now it looks good now let's just copy this for
5:43:35
the delete icon and we'll change the icon name to
5:43:40
delete and we're going to use the rose color from our theme like this so it looks good so if I open any other post
5:43:48
we won't see this because this is not my post and this works fine now I want to
5:43:54
show a delete Model when we click on the delete icon so let's add a handle post
5:43:59
delete function and this will just confirm the user if he really wants to delete this post so let's create this
5:44:05
function here and inside this we're going to use the alert so let's just use the same alert
5:44:12
we used for deleting the comments so let's go into the comment item and here let's just copy this
5:44:22
alert and paste it here let's import the alert it's already imported and we don't
5:44:29
really need to change anything because we will just call this on delete function and pause the item so we
5:44:36
already get getting this function from the parent component here so everything
5:44:42
looks all right so let's just save this and test this so let's open the terminal
5:44:48
and now if I click on this delete icon and hit delete uh we see this delete post console log from our parent
5:44:54
component and if I click on the edit um I think this is the event when we
5:45:03
click on any button so yeah here we need to pause the item
5:45:08
so let's just add a function which will call this on delete function and we need to pass the item this will be the post
5:45:15
so let's save this and now if we click on the edit icon we see this console log from the par component so everything's
5:45:22
working great now let's go into the parent component post
5:45:27
details this one and now we need to call the APA when we hit the delete icon so
5:45:33
let's just add a comment here delete post here now let's go go into our post
5:45:38
service and here let's just duplicate this remove comment function and let's change the name to remove post this will
5:45:46
save the post ID and we need to delete the data from the post table so let's change this from
5:45:53
comments to posts and let's add the post ID here now
5:45:59
let's change this error to remove post error and the message will say could not remove the post so let's change this
5:46:09
and then we just need to return the post ID and the data like this so this function is done now let's go into the
5:46:16
post details and here let's just call this function and this will give us a
5:46:23
response so let's use the remove post function and we need to use the item. ID
5:46:30
as the parameter but we can also use the post data since we have the post object in the post details component
5:46:37
so either way it's going to work the same now if the result. success is true that means the post is deleted and in
5:46:44
that case we'll just go back to the previous screen like this and if the success is false that means we have an
5:46:51
error so let's use an alert to show this error to the
5:46:57
user like this and let's just save this so now let's see if this is working so
5:47:03
I'm just deleting this test post so we go back and that means the post is
5:47:09
deleted but we still see this post in the home feed and this will be fixed later but for now let's just refresh the
5:47:15
app and now if I scroll down we won't see this post okay so the post is gone now I want
5:47:24
to test this again so let's create a new test post and hit
5:47:32
post um cannot convert undefined value to object
5:47:37
okay I know why this error occurred so when we create a new Post in the post listener we update the post State and
5:47:45
that's where we need to add an empty array of the comments and the likes so that's why it's causing this error so in
5:47:52
this handle post event we need to add an empty array of post likes for this new post and we need to do this for the
5:47:59
comments but for comments we need to follow the same syntax we using in the postcard component so here we need to
5:48:05
add an array with an object and the property will be count as zero so let's
5:48:11
save this and hopefully this won't happen again so let's create a new test
5:48:16
post test two and hit post so there we can see our new post so
5:48:24
this is working fine now let's open the post details and now if I click on delete if we go back that means the post
5:48:31
is deleted but we have to refresh the app to see the updated home feed like this but we need to fix this and we will
5:48:38
add one more condition inside this function and we will check for another event so if the payload do event type
5:48:46
equals to delete that means a post is deleted and in that case we'll just
5:48:51
update the post state so now let me just console log the payload and see what do
5:48:56
we get when we delete a
5:49:03
post let's save this and now let's try to delete this this
5:49:11
post so the post is deleted and we can see in the console event typ is deleted
5:49:16
and we have the post ID in the old object so this is the deleted post ID and we can use this to add into the
5:49:23
condition so if the payload do event is lead and we have the payload do. ID then
5:49:31
uh it means we deleted a post and we need to update the post state so let's
5:49:36
use the set post function and this will give us all the previous post in a call
5:49:41
back so let's create the updated post and we will filter out all the post
5:49:46
where the post ID is not equal to the one that we just deleted so post out ID
5:49:51
is not equal to the payload do old.id like this and then we just return
5:49:58
the updated posts now let's save this and see if
5:50:06
this is working so let's just refresh the
5:50:11
app and let's create a new
5:50:18
post so there we can see our post and let's open the post details and now if I delete the
5:50:25
post we go back and now we don't see the deleted post and the home feed is
5:50:30
updated as well so let's just comment this and now we're going to work on edit post so if I open the post details
5:50:38
uh this is not my post
5:50:43
so actually let's just create a new post from this account and then we will edit
5:50:48
that so let's include an image let's use this one and then add a caption as on
5:50:54
the top of the word then hit
5:51:00
post so there we can see our new post and if I open the post details there we
5:51:06
can see the actions and now we're going to work on edit action so let's go into
5:51:11
the post card so the idea is when we click on this icon we need to move to
5:51:16
the new post screen where we'll just fill all the existing post data um we need to go into the post
5:51:24
details and then from there user will be able to update this post so on this
5:51:29
function we need to use the router and then push a new route so the name will be new post and
5:51:38
for the parameters we'll just separate the item so this will be the post data on this new post
5:51:43
screen so let save this and now if I click on the edit icon we will move to
5:51:49
the new post screen actually we do move to the next screen but for some reason
5:51:54
this model is not dismissed so to fix this we're going to move back from this
5:51:59
model and then to this new screen so here let's use router. bag so this will
5:52:05
dismiss the model and then we we will move to the new post screen so if I open the post details and click on the edit
5:52:12
we move to the new post screen and the model is dismissed so now we need to show this post data that we passed from
5:52:19
the previous component and to be able to fetch this data we will use a hook called use local search prams uh this
5:52:27
will get us all the parameters that we paed from the previous component so let's just console log it and see if we
5:52:33
got this data so in the console we can see we got this post data that
5:52:39
includes the body the comments and the file so we only need the body and the file to fill this data now if I open the
5:52:47
new post screen from the home screen we won't see this because we didn't pass the post data let me just remove this
5:52:53
console log and there we can see the post is empty object now let's go back to open
5:53:00
post details hit edit there we can see the post data now we need to fill this
5:53:05
data and for that we need to define a use effect hook so let's create a hook
5:53:14
here then let's add an empty dependency array and first we will check if we have the post from the previous component so
5:53:20
this will be true and in that case we will set the body reference. current value to the post body so we have this
5:53:28
reference to hold the post body and this will be post. body then we need to set the file if we
5:53:35
have any so uh post. file or null if we don't have any
5:53:40
file so let's save this and we don't see the editor content but we do see this
5:53:46
icon and there we can see the image so this image is coming from a remote URL
5:53:52
so there we can see we used a function get superbas file URL and this is being
5:53:57
used to uh show this remote file so now we're going to update the
5:54:02
content inside the editor and to update this we're going to use the editor reference so let's use it here editor
5:54:10
reference do current then we can use a function called set content
5:54:17
HTML then we need to pause the post body here and let's save this so there we can
5:54:24
see the content is updated but we have an issue with this so if I open the post details and hit
5:54:31
edit we can see the image is updated but the content is not updated and that's
5:54:36
because uh it takes some time to load this Library so we're going to use the set timeout function to update this
5:54:44
content editor so we will use 300 milliseconds and then we will update the content of this editor so now if I go
5:54:51
back and open the post details hit edit so there we can see the content is
5:54:57
updated and the image is updated so now we need to change this button to update
5:55:03
when we have the post data so let's use a condition as post and post. ID in that
5:55:09
case this will say update otherwise it will say post like this and let's save
5:55:15
this so there we can see it's saying update because we are updating this post so now let's see what do we need to
5:55:22
change in the API so let's go into this function and actually we don't need to
5:55:28
edit anything here we using upsert function and this handles the inserting and updating as well and the only thing
5:55:35
we're going to do is include the post ID when we are updating so here after
5:55:41
defining the data we will add a condition if we have the post and the post ID that means we are updating this
5:55:47
so we need to include the ID into the data so now it will hopefully update the
5:55:53
post instead of creating a new one so let's save this and see if this is
5:55:58
working so uh let's just update the post content to updated
5:56:06
and then hit update so we go back that means it
5:56:11
worked but we don't see the changes here so let's open the post details and there we can see the post content is updated
5:56:18
now let's edit this again and this time we're going to update the post file so let's just remove this one and choose a
5:56:25
new image uh let's choose this one and then hit
5:56:33
update so we go back and it's updated but now let's open the post details to
5:56:38
see the updated post and there we can see the content is updated and the image is updated but
5:56:45
this is not updated in the home feed that will'll fix in a second now let's create a new test post and hit
5:56:54
post then let's just refresh the app and now we will see the updated changes in
5:57:00
the post now to fix this we will have to listen for the post updates in the home
5:57:05
screen so let let's go into the home screen and here uh where we are listening for the Post events we will
5:57:11
just copy this insert event and paste it here let's change this to update and
5:57:16
whenever a post is updated this will be triggered and we will get the updated data in the new object and we need to
5:57:23
update the post so this will give us the previous post within a call back now let's define the updated
5:57:31
posts so we need to use map function on this post and uh for each post we will
5:57:37
have a condition if the post ID equals to the payload do new. ID so if this is
5:57:44
true that means this post is updated and in that case we will just update the post body and the post file from the
5:57:52
payload do new. body and the file like this now let's return this
5:57:58
post and then we also need to return the updated post as
5:58:04
well so let's save this and see if this is working now we have this test post so
5:58:10
if I open the post details then hit edit and let's CH this to testing post
5:58:18
update then hit update so there we can see the post is
5:58:24
updated in the home feed as well and if I open the post details we can see the updated post data
5:58:31
now let's edit this one more time and this time we will include an image
5:58:36
so let's choose these floors and let's also change this post
5:58:43
body to colors like this and then hit
5:58:51
update so now we can see the body is updated and we see the new image so post
5:58:56
update is working as well so with this remove and edit posts are done now we're
5:59:01
going to work on the user profile section and fetch the user post and then implement the post pagination let's go
5:59:08
into the user profile and here the bottom section looks very empty so let's go into the profile section and here
5:59:16
we're using this user header component and the only reason to create this is because now we're going to use flatlist
5:59:23
and this can be used as a flatlist header component now so we're going to have the same structure as the home
5:59:29
screen so let's just copy the States from this component and and then paste it
5:59:38
here then we also need a limit variable so let's add it here and give this a
5:59:43
value of zero now let's go into the home screen and here let's just copy the get post
5:59:50
function this is the one so let's just copy this and then paste it
5:59:57
here so we don't need to change anything so let's import a fetch post
6:00:03
function and this will fetch the post and set them into the state now we also need to pass one more parameter as the
6:00:10
user ID and we have the user object from our use Au hook and uh let's just save
6:00:17
this uh use T doesn't exist so let's just import it and save this now if I go into the
6:00:26
profile we don't see any error so and now we need to go into the fesh post
6:00:32
function and here let's receive this user ID so here based on this user ID we
6:00:37
need to fetch the post for only that user so let's just copy this code and
6:00:43
here we'll make a condition if we have the user ID then we'll fetch the user post but if we don't have the user ID
6:00:50
then we'll fetch all the posts like this so we'll use the same code there won't be any drastic changes but we will only
6:00:58
just add one condition here so let's use a function
6:01:03
EQ and this will make a condition if the user ID equals to this user ID that we
6:01:09
passed and this will fetch only that users's post and we will show it in the profile section now it should be
6:01:16
fetching all the user post and setting them in the post trade so now let's use a flat list component and show these
6:01:22
posts here let's add the data as the
6:01:30
posts um actually instead of writing all the properties let's just copy the flat
6:01:36
list component from the home screen because all the properties will be the same so let's copy this and paste it
6:01:44
here and let me just format
6:01:50
this okay so the data is the post and then vertical scroll bar is hidden we
6:01:55
have the list style and the key extractor is the ID now we need to import this post card
6:02:03
component from the components and here we are posing the item and the current
6:02:08
user so make sure you import this current user from the use Au hook uh yes
6:02:13
we have imported then we have the router and then we calling this get post
6:02:18
function when we reach the end of this flat list so let's import this loading
6:02:24
component and this will say no more post when we have fetched all the post so let's save this and see if this is
6:02:31
working so there we see the loading and the header component at the bottom
6:02:36
and now we can see all the post from this account and the pagination is working as well so now this is working
6:02:43
but we need to move this user header to the top of the screen so let's just copy
6:02:48
this and in the flat list we can define a list header so let's use list header
6:02:54
component and we're going to use the user header here so let's save this so
6:03:00
now if I scroll to top we see our header now we need to add some space between the content and the header and we can
6:03:07
use a property as list header component style so let's give it a margin bottom
6:03:12
of 30 pixels and let's save this so now we have some space at the bottom now
6:03:17
let's just refresh and see the profile again so this is our home feed now let's
6:03:23
go into the profile we see the loading and then we see all the post
6:03:29
from this account only and the pagination is working as well so if I scroll down it's fetching more more
6:03:36
posts and if I scroll all the way down we see no more posts so now I want to
6:03:42
change a few of the things so I want to decrease the space on the top when we have when we don't have any post so
6:03:48
let's change this to 100 pixels and by default we are fetching four post so
6:03:53
let's change this to 10 and let's also change this to 10 in the home
6:03:59
screen so now it will fetch 10 post on the first try and when we scroll down on
6:04:04
the second time it will will fetch 20 so we can like the post or let's try to
6:04:10
create a comment test and hit send there we can see the new comment
6:04:17
and if I try to edit the post this works as well and everything is working
6:04:23
great now let's go to our document and we have done user post and post
Notifications
6:04:29
pagination and let's see if anything is still pending uh we have implemented Ed the
6:04:36
post like as well and I think I forgot to add the steps for notification screen
6:04:42
but anyway uh let's just create a mockup for that screen so let's Pi a rectangle
6:04:47
and this will be our notification screen then on the top left corner we will have
6:04:52
an arrow icon to go back to the previous screen so let's add a back arrow here
6:04:58
now after this we're going to show the notification items so let's just copy the design for the comment item because
6:05:05
we got use the same design here so let's just paste it here let's make it big now
6:05:11
we're going to store only the comment notifications so this will say commented on your post something like this and
6:05:18
here will show the username and the user Avatar so this will be our notification screen we're also going to use
6:05:25
notification count and we're going to use real time updates for that but that will come later so let's go to our
6:05:33
notification screen and we can access it using this hard icon now let's just
6:05:38
refresh the app and uh let's open a post let's open this one and the idea is we
6:05:45
will send a notification to the post owner whenever a comment is created so let's go into the post details and here
6:05:52
when we create a new comment here if the success is true then we will create the notification so let's just go to our
6:06:00
services and create a new service as notification service
6:06:06
and let's go to our post service and here we'll just copy an existing function uh let's copy this one create
6:06:13
post like and paste it here and let's change the name to create notification we will
6:06:20
receive the notification object and this will be saved into the
6:06:26
notifications table so let's change this to Notifications uh we will insert the data
6:06:33
and uh everything else is fine now let's change this to notification error and
6:06:38
for the message um let's just change this to something went
6:06:45
wrong like this so let's save this and now we can use this when creating a new
6:06:50
comment so let's go into the post details and here if the success is true
6:06:55
only then we will send the notification but we don't want to send the notification if the post owner comments
6:07:01
on his own post so I will make a condition if the current user ID is not equal to the post. user ID which is the
6:07:09
owner only then we will send the notification so now let's create the
6:07:14
data for this new notification and this will include the sender ID which will be the current user
6:07:21
ID so let's use user. ID then we will have the receiver ID and this will be
6:07:27
the post owner so post. user ID and let's add the title as commented on your
6:07:34
post we also want to add some extra data so
6:07:39
this will be the post ID and the comment ID this will be the Json object but we
6:07:44
want to stringify it so that we can save it into our database so post ID like
6:07:50
this and then the comment ID um okay so this should be colum like
6:07:57
this so we can get the comment ID on the result so when we create the new comment
6:08:03
uh in this API uh we receive the newly created data so and this include the ID
6:08:08
of the comment as well so result. dat. ID like this now let's pass this data to
6:08:14
this new function create notification and let's save this now
6:08:22
this will hopefully create the notification when we create a new comment so let's test this out so
6:08:29
cute and hit send uh I think we have an error
6:08:35
so it says notification error soab base doesn't exist so we need to import the soab base client into this file that was
6:08:43
the error so let's save this and test this
6:08:49
again let's hit send so the comment is created but we
6:08:56
don't see any console log because we never console logged anything so let's just go to super base and see if the new
6:09:04
comment is created okay so there we can see a new comment
6:09:09
and we have the sender ID receiver ID post ID as 12 and the comment ID as 9 so
6:09:15
if I go to the comments we can see we have a new comment with the post ID of 12 and the comment ID of nine so
6:09:22
everything is working fine now let's just close this and uh let's create a
6:09:29
new comment on this post and hit send
6:09:37
so there we can see a new notification should have saved into the database now let me just uh log out from this account
6:09:44
and we will log into a different account where we can see the notifications and then we will Design our notification
6:09:51
screen so let's log to this account and then hit
6:10:01
login not now Okay so we logged in and now let's
6:10:06
go to the notification screen so before we start working on the design I want to
6:10:11
create an API to fetch all the notifications so let's go into the post service and let's just copy this
6:10:18
function fetch post details and then paste it
6:10:23
here so let's change the name to fetch notifications and this will see a
6:10:28
parameter as the receiver ID and then let's change the table name to notifications
6:10:35
so when we specify staric that will mean it will fetch all the data from the notifications but I also want to fetch
6:10:41
the sender data as well so if you remember previously we added two relationships inside the notification
6:10:47
table uh we have the sender ID and the receiver ID connected to the users table
6:10:53
so in our case we want to use the sender ID forign key and I want to get the ID
6:10:59
name and the image of that user so we'll specify that here and here we want to
6:11:05
make a condition if the receiver ID is equal to the receiver ID that we pass to
6:11:11
this function now let's remove this foreign table from the order function and let's also remove this single
6:11:18
function and now let's change this fetch post details error to fetch
6:11:24
notifications error and the message will say could not fetch
6:11:30
notifications like this so everything looks fine so let's go into our
6:11:36
notification screen and here first let's create a state to store all the
6:11:42
notifications and by default this will have an empty
6:11:47
array then we will call a function inside the use effect hook to fetch all the notifications so let's add an empty
6:11:55
dependency array and we will have a function get
6:12:02
notifications so let's define this function here this will be an async
6:12:08
function and here we will get the notification so let's get the result from this API now we need to pass the
6:12:16
receiver ID and that we can get from the current user so let's get the user from use OD
6:12:23
hook so we want to fetch all the notifications for this user so let's pause this ID and let's just console log
6:12:31
the result and see if you're getting the data
6:12:36
let's save this and there we can see a new console log in the terminal so we
6:12:42
have notifications as data and success is true currently we have only one notification this includes the sender
6:12:48
object as well so let's just remove this console log and here we will check if
6:12:54
the result. success is true then we will set this data to notification State like
6:13:01
this so let's just minimize this and now I'm just going to to copy and paste all
6:13:06
the stylesheet for this notification screen and then I will show you the Styles whenever I use them so these are
6:13:14
pretty basic Styles now let's just change this to our screen wrapper component and here we will have a view
6:13:21
with a style of container and you can see we have a style of Flex one and some padding
6:13:28
horizontal then inside this we will have a scroll View and let's just hide the
6:13:34
vertical scol indicator and this will have a container style of list
6:13:41
style and for the list style we have the padding vertical and a gap of 10 pixels
6:13:47
now uh let's display the notifications here so let's use the map function this will give us the item in the call back
6:13:54
and we will return a new component so let's call this notification item first
6:14:00
let's pause the item and then we will also have the key as the item.
6:14:06
ID and we also need to pause the router because we want to open the post details
6:14:11
from the notification item so let's just declare a router here let's use use
6:14:17
router hook like this and now we need to create this function so let's create a
6:14:23
new file notification item and then create a functional
6:14:28
component let's save this and let's just import it here and let's save this
6:14:35
um we forgot to import this WP and HP function so let's import them and import
6:14:41
the theme object as well and now let's save this so there we can see a
6:14:47
notification item because we only have one notification for this user so if we
6:14:53
don't have any notification I want to display a message so let's create a new condition here we will check if the
6:14:59
notifications. length equals to zero then we will display this message so let's use text component and give this a
6:15:06
style of no data and the message will say no notifications
6:15:14
yet and let's save this so now if I just don't set the data inside this function
6:15:21
and let's just refresh the application and let's go to notification screen there we can see no notifications
6:15:29
yet um we also forgot to add our header so let's import the header component
6:15:36
and let's give it a title of
6:15:41
notifications let's close it and save this so there we can see our header now
6:15:47
let's just UNC commmand this and there we have the notification item so let's go into the notification item and first
6:15:54
of all I'm just going to copy and paste the stylesheet for this component so now let's get the item and
6:16:02
the router property that we paused from the parent component now let's change this View to a
6:16:07
touchable opacity and let's also give this a style
6:16:15
of container and let's save
6:16:20
this uh we need to import the theme object also you can just copy the styles
6:16:26
from here and let's import the HP and WP functions then let's save this so there
6:16:33
we can see the notification item style now let's add the on press property on
6:16:39
this button and we will trigger the handle click function so let's define it
6:16:44
here and basically this will open the post details whenever we click on any notification item but for now let's just
6:16:51
add a comment open post details now let's just remove this text
6:16:58
and here we going to use the Avatar component so let's import it and we're going to show the sender image here so
6:17:04
so if I just comment this and let's just console log the item
6:17:14
here and let's save this so there we have a new console log in the terminals
6:17:19
so we have the notification item and here we should have the sender object so
6:17:25
there we can see we have the sender data and we have the image property so let's
6:17:30
just comment this and now we're going to pass the UR of this cender image so URI
6:17:35
will be item. sender. image like this then let's add the size of
6:17:42
hp5 let's close this and save this so now we can see the user image also these
6:17:49
are the container sty so you can just pause the screen and copy them now let
6:17:54
me just format this and let's create a view and give
6:18:00
this a style of name and title and in this view we're going to show the name name of the user and the title for the
6:18:06
notification so we have the following styles for this one now let's create a
6:18:11
text and give this a style of text then inside this we're going to
6:18:18
show the user of the name of the sender so item. sender. name I save this so
6:18:26
there we can see the sender name and let's just copy this for the title of this notification so this will be item.
6:18:34
tile so let's just remove this let's save this and there we can see the title
6:18:39
now I want to make this title a bit darker so let's add the color property
6:18:44
as the text dark color from our theme let's save this and now it looks
6:18:51
good so next I want to show the time of this notification on the right side so
6:18:57
let's create a text after this View and give this a style of
6:19:03
text and and this will have the light color so let's add the color property as
6:19:08
the text light from our
6:19:14
theme and then let's add item do created add property here let's save
6:19:21
this um this should be Styles let's save this and let's go to
6:19:28
Notifications so as you can see the date doesn't look good and we're going to format this using the moment Li Library
6:19:35
so let's create a new variable here created add then let's import the moment
6:19:40
Library let's pass this item. created add
6:19:46
property then we're going to use the format function and we're going to have a format of Triple M and then
6:19:53
D and let's use this property here let's save this and now date looks
6:20:00
good so now we're going to open the post details whenever we click on the notification item so let's go into this
6:20:07
function and here first of all we're going to get the post ID and the comment ID from the notification data so if you
6:20:14
remember previously we stringified this data and stored it into the database now
6:20:20
we have to pause this data and we will get the post ID and the comment ID so
6:20:25
here you can see we have the post ID and the comment ID as a string now let's use the router and then navigate to the post
6:20:32
details and then we also going to pass the
6:20:37
parameters as the post ID and the comment
6:20:44
ID like this and let's save this so now if I click on this notification item it
6:20:50
opens the post details model and it fetches the post data and shows the comment list so this is working great
6:20:57
but now I want to make it so that when we click on a notification it should highlight the comment that was related
6:21:03
to the notification so for that we need to go into the post details and here we need to get the
6:21:09
comment ID from the local search parameters like this so now we can use
6:21:14
this to highlight the comment so let's go into the comment section and for each comment item we will pass a new property
6:21:22
as highlight and we will check for the comment ID which will match this comment ID that we got from the previous
6:21:28
component and if this matches then the Highlight will be true and based on this we will Style the component differently
6:21:36
so let's go into the commit item and receive this property by default this will be false
6:21:43
but whenever this is true we will add one more style object to this container so let's add an array and when this
6:21:51
highlight value is true then we will pass the Highlight Styles let's save this and for the
6:21:58
Highlight Styles object we have the following Styles we have some border and Shadow properties so you can just copy
6:22:05
them so now if I click on this notification item you can see the first comment item is highlighted and that's
6:22:12
very useful because user can know which comment is related to which notification item so if we open the post details from
6:22:20
the home feed you won't see this highlight because highlight is by default false so everything is working
6:22:27
great now we want to show the notification count on this hard icon so for that we will add a realtime data
6:22:34
updates in the home screen so here in the use effect let's create a new notification
6:22:40
Channel um let's just copy this one and paste it here and we'll change the name to
6:22:47
notification Channel like this and let's change the channel to
6:22:53
notifications and we only going to listen for the insert events so let's change this to insert and schema will be
6:23:00
public and the table will be notifications then I also want to use a filter because we want to fetch all the
6:23:07
notifications where the receiver ID is the current user ID so let's go into the
6:23:13
post details and here we already have implemented this filter so let's just copy this and then paste it here so we
6:23:21
will apply a condition where receiver ID is the user
6:23:27
ID so user. ID then let's change this function to
6:23:33
handle a new new
6:23:39
notification now let's create this function
6:23:49
here so this will receive the payload that will contain the new notification data as well so let's just console log
6:23:56
this payload and this will say got new notification like this
6:24:05
so now we want to remove this listener whenever this component unmounts so uh
6:24:10
where we return this call back let's just copy this line and let's pause this notification Channel and let's save this
6:24:18
so now we need to test this and for that we have to put both the phones side by side so that if we create a comment on
6:24:25
the first phone and then on the second phone we can receive this notification so let's reload the application on the
6:24:32
Android there we can see all the post and I believe this phone is using my account
6:24:39
yes and on the iPhone we logged in with a different account so now let's try to
6:24:45
create a comment and see how it goes let's open the post details and here let's create a
6:24:53
comment um cannot read the property ID of null from
6:24:59
user I think the session got expired for the Android app so let's just
6:25:06
reload and and we move to welcome page so obviously the session was expired so let's log in with Jon Snow's
6:25:20
account then hit login okay so now we logged in with Jon
6:25:26
Snow's account on this Android device so let me just confirm by creating a new
6:25:32
comment let's see if we get this error now okay so now we got a new command and
6:25:40
this is working now let's create a comment on the second users's post so
6:25:45
let's go uh here we have the second user post let's open the post details and
6:25:51
here let's create a new comment hit send and now let's see in the console we
6:25:58
should have got the payload so there we can see got new notification so now we
6:26:04
going to create a notification count that will increase every time whenever this current user receives a
6:26:10
notification so let's create a state notification count and set notification
6:26:17
count by default the count will be zero and in the event listener here we
6:26:23
will make a condition so if the payload do event type is
6:26:29
insert and we have the ID value inside the payload do new object that means we
6:26:36
got a new notification and in that case we will set the notification count so
6:26:41
previous notification count + one like this and let's save this now we're going
6:26:47
to use this to show the count on the heart icon so let's just minimize this
6:26:52
and let's go to the section where we use the hard icon so here we need to make a condition
6:26:59
if the notification count is greater than zero only then we will will add
6:27:04
this count so let's create a view and give this a style of
6:27:10
pill then inside this let's create a text and let's give this a style of
6:27:16
build text and inside this we will use the
6:27:21
notification count and let's close this and save this
6:27:27
so for the pill and the pill text we have the following Styles so you can just copy them we have a position of
6:27:33
absolute and this will display on the top right corner of this hard icon so
6:27:39
now if I create a new comment and hit send there we can see the new comment
6:27:45
and we also see this notification count on the hard icon of the second device
6:27:50
and because we increase the count value that's why we can see this on the heart icon so if I open this we see the
6:27:57
notification and if we open this first notification we can see this is the comment that was made so this is
6:28:04
highlighting the comment now we're going to try to create a comment on the iPhone and we'll see if
6:28:11
we can get the notification on the Android so let's open a post from Jon
6:28:17
Snow uh let's open this one let's create a comment and hit
6:28:24
send so there we can see the new comment and we also see this notification count
6:28:29
so if I click on this we see all the notifications and if I open the first one
6:28:35
here you can see we made this comment so this is highlighted so let's just refresh the
6:28:41
application and we have one more thing left to do so when we have the notifications and we click on the heart
6:28:47
icon it should remove the notification count so right now if I create a
6:28:54
comment let's create a test comment on this post there we see the count and if I
6:29:00
click on it it should reset the count to zero so let's go to the hard
6:29:08
icon and here let's just change this to a function body and first we will set the
6:29:15
notification count as zero then we will move to the notification screen so that
6:29:20
save this so now if I open the notification screen and move back we
6:29:26
don't see the notification count because it's it's reset to zero so let's just reload the
6:29:33
application and and finally we have completed this application so let's just
Final Demo
6:29:38
do a final demo of the whole application and see what features we have built so let's go into the profile and let me
6:29:45
just log out so the first thing you will see when you open this application is this
6:29:50
beautiful welcome page and you can sign up or login from this so I'm going to sign up so let's create a new user as
6:29:59
Sarah let's add a password and hit sign
6:30:05
up so once you log in you will be redirected to this homepage now let's go
6:30:11
into the profile and let's fill the details for Sarah let's add a random phone number she lives in London and
6:30:19
let's add a bio as cooking and hit update uh we need to add the profile
6:30:25
pictures because this is required so let's use this one and then hit update
6:30:34
so now the profile data is updated now we're going to create some post from this account so let's go to the homepage
6:30:41
and on the homepage we can see all the post from all the users and as I scroll down you will see more and more post
6:30:49
will be fetched so let's go to the new post screen and here we will create a new post as hey guys I'm new
6:30:58
here and let's include an image and let's choose this cute girl
6:31:04
then hit post so you will see as soon as the post
6:31:10
is created it will appear on both of the devices because of the real time changes
6:31:15
so let's create a comment from Jon Snow's account on this post hey Sarah
6:31:22
welcome and let's hit send so there we can see the new comment
6:31:28
and a new notification item so if I click on this notification have we see this was the
6:31:34
comment made from the John son account so Sarah can see all the notifications from all his posts then we can like any
6:31:42
post so if I like this one uh this won't reflect on the home feed because we haven't used the real time updates for
6:31:50
the likes but you can just add a new channel here listen for the likes and then update the home feed and this will
6:31:57
just work so let's just reload and there we can see the like so I can like this
6:32:04
one or I can create a comment on this one so if I create a comment on this one
6:32:09
this also won't reflect in the home feed and I purposely left this feature
6:32:14
because you guys can try this and add these features and see how everything is
6:32:20
working so as I scroll down this will fetch more and more posts and we have
6:32:25
the video post as well so I can play pause this video I can move forward and
6:32:31
I can make it full screen so this is a fully fun functional video player and it's working perfectly fine now if I
6:32:38
scroll all the way down we should fetch all the post and at the last it should display a message no more post like this
6:32:46
and the same behavior is on the profile screen so here you can see all the user
6:32:51
post and here the pagination works as well then we also implemented some post
6:32:57
and the comment actions so if you have created a post or the comment you will see the delete or edit icon so let's
6:33:04
open the post where we commented from Jon Snow's account let's open this one
6:33:10
and here I can just delete this comment because I made this or the post owner
6:33:16
can delete this so now if I open the post details from here you can see all
6:33:22
these comments are gone now let's try to edit a post and
6:33:27
see if this works and let's create a new post from this new
6:33:32
account and z family time and let's include an image let's
6:33:38
choose this one and then hit
6:33:45
post so there we can see this new post and now if I open the post
6:33:52
details and then hit edit icon this will open into the same screen and I will
6:33:57
have all the content as it is now I can choose this and change this to quality
6:34:02
time and and let's make it board and now if I update this this will be updated on
6:34:09
the both devices because of the real-time changes we implemented so this works great now let me show you our
6:34:17
powerful re text editor we have all these options to style our content so we
6:34:22
can add a list one two three and let's just remove this
6:34:28
and then we can add something and if we want to start it we can move it around
6:34:35
into the center make it bold italic and let's just remove this and make it big
6:34:41
using this H1 or H4 then I can move it around and let's just leave it like this
6:34:48
and if I post this you will see it will be posted as it is with all the Styles
6:34:53
so all the users can see this and uh let's edit
6:34:58
this and let's change this to hi everyone
6:35:04
and then hit update and this will be updated for all the users who are using
6:35:10
this application now let's create a comment as hey and hit
6:35:17
