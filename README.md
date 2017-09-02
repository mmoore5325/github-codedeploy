# appspec notes

APPSPEC.YML PROPERTIES
==================================================================
*** means THIS IS NOT PART OF THE FILE BUT A DESCRIPTION OF WHATS BELOW
==================================================================
SPACING MATTERS IN THIS SIMILAR TO EXPRESS, EVERYTHING WITHOUT *** HAS TO BE IN THAT EXACT ORDER.  FOR INSTANCE, "files:" IS UP AGAINST THE LEFT WALL NO SPACES, AND BELOW IT, "  - source:" IS RIGHT BELOW IT, 2 SPACES WITH A HYPHEN/DASH, A SPACE, THEN THE WORD SOURCE.  BELOW THAT, DESTINATION, IS 4 SPACES EXACTLY.  INCORRECT SPACING WILL BREAK YOUR APPSPEC FILE!!!
==================================================================

version: 0.0
os: linux (or "windows")

*** FILE SECTION OPTIONAL

files:
*** LINUX SOURCE
  - source:  myfolder/myfile
    destination: /myfolder/destinationfolder/
*** WINDOWS SOURCE 
  - source: myfolder/myfile
    destination: C:\myfolder\destination\

*** PERMISSIONS SECTION OPTIONAL

permissions:
*** OBJECT IS ONLY REQUIRED PROPERTY

*** SPECIFIES PARENT FOLDER DIRECTORY THAT YOU WANT TO TAKE ACTION ON
  - object: /myfolder

*** PATTERN IS OPTIONAL, REGULAR EXPRESSION TO FIND ONE OR MORE FILES YOU WANT TO MAKE A PERMISSION CHANGE ON.  ONE BELOW UPDATES ALL JAR FILES
    pattern: *.jar

*** EXCEPT IS THE OPPOSITE OF PATTERN.  UPDATE PERMISSIONS ON ALL FILES "EXCEPT" THE REG EX SPECIFIED, ONE BELOW UPDATES ALL JAR FILES EXCEPT ONES THAT END IN "-test"
    except: *-test.jar
***  UPDATES/SPECIFIES OR CHANGES OWNER, SAME SORT OF OWNER GROUP WHEN DOING A CHMOD COMMAND
    owner: ec2-user
*** UPDATES/SPECIFIES OR CHANGES GROUP
    group: minedminds
*** TYPICAL OCTAL MODE SET FOR PERMISSIONS
    mode: 644, 755, 4755 (*** JUST USE ONE OF THESE, THESE ARE EXAMPLES ***)
*** ACLS SPECIFCATIONS, THE ONES BELOW GIVE THE USER AND GROUP MENTIONED ABOVE READ WRITE AND EXECUTE PERMISSIONS
    acls:
      - u:ec2-user:rwx
      - g:wheel:rwx
*** CONTEXT FOR THOSE OF YOU USING SE LINUX - NO EXAMPLES GIVEN
    context:
      user: user-specifcation
      type: type-specifcation
      range: range-specification
*** TYPE ARGUMENT ONLY TAKES 2 POSSIBLE VALUES, EITHER FILE OR DIRECTORY
    type: 
      - file
*** IF YOU SET IT TO FILE, IT WILL BE BASED ON THE OBJECT/PATTERN/EXCEPT COMBO SPECIFICATION MENTIONED ABOVE
*** IT ISNT GOING TO CHANGE THE MYFOLDER DIRECTORY PERMISSIONS, ONLY THE FILES CONTAINED WITHIN
*** IF YOU WANTED TO UPDATE THE DIRECTORY AND ALL CHILD DIRECTORIES AND OBJECTS, DELETE PATTERN AND EXCEPT AND IT WILL MODIFY ALL FILES WITHIN THE DIRECTORY INSTEAD OF INDIVIDUAL FILES
*** EVERYTHING MENTIONED IS FOR LINUX ONLY AND NOT AN OPTION FOR WINDOWS, WHY ARE YOU USING WINDOWS ANYWAYS
    type:
      - directory 

*** THAT IS THE END OF PERMISSIONS

*** THE NEXT SECTION IS HOOKS
*** THERE ARE NINE(9) LIFE CYCLE EVENTS GO IN THIS ORDER, ONES NOT IN COMPLETE CAPS ARE REQUIRED BY AWS
*** THE "Start", "DownloadBundle", "Install", and "End" events in the deployment cannot be scripted.  However, you can affect what's installed during the "Install" event, and where,
*** by editing the "files" section of the AppSpec file.
START --> ApplicationStop --> DOWNLOADBUNDLE --> BeforeInstall --> INSTALL --> AfterInstall --> ApplicationStart --> ValidateService --> END

Start is exactly what it sounds like
DownloadBundle is where code deploy agent attempts to download the code deploy revision or bundle specified
Install is the lifecycle event where the files in the file section, if you've specified any, get copied
End is where the code deploy agent sends a success signal to the code deploy endpoints to let the code deploy service knows it successfully completed deployment.

*** FOR THE LIFECYCLE EVENTS WE CAN ATTACH SCRIPTS TO, THEY HAVE MEANINGFUL NAMES AS TO GIVE AN IDEA AS TO HOW TO BEST USE AN ORGANIZE, BUT ARE ABITRARY AS YOU CAN
*** ADD WHATEVER SCRIPT YOU WANT INTO WHATEVER LIFECYCLE EVENT YOU WANT
*** THERE IS ONE CAVIATE IS ONE LIFE CYCLE EVENT SLIGHTLY DIFFERENT THAN THE OTHERS, "APPLICATIONSTOP"
*** THE SHORT VERSION IS THE APPLICATION STOP SCRIPT THATS CONNECTED TO THE LAST SUCCESSFUL REVISION IS THE ONE THAT IS GOING TO BE EXECUTED
*** THE LOGIC FROM AWS IS THAT YOUR CODE DEPLOY PATCH CHANGES THE WAY YOUR APP WORKS INCLUDING THE WYA IT STOPS, YOU DONT WANT TO RELY ON YOUR CURRENT BUNDLE CHANGE, BUT YOUR LAST BUNDLE

*** ADVICE IS TO START WITH "BeforeInstall" rather than "ApplicationStop", as it always looks in the current codedeploy revision, rather than applicationstop, which looks in the last code deploy revision.

hooks:
  BeforeInstall:
    - location: scripts/httpd-stop.sh (FOR EXAMPLE ONE THAT TURNS OFF APACHE OR IAS)
      timeout: 300  (***TIMEOUT IN SECONDS, IF NOT SPECIFIED, BY DEFAULT, ITS 3600 SECONDS, WHICH IS ALSO THE MAXIMUM AMOUNT OF TIME YOU CAN GIVE, AND IT REPRESENTS THE WHOLE BEFOREINSTALL EVENT, NOT THE SCRIPT ITSELF)
      runas: root (***ONLY APPLICABLE IN LINUX)
*** THE INSTALL IS AUTOMATICALLY RAN BETWEEN BEFORE INSTALL AND AFTER INSTALL AND WILL COPY ANY SPECIFIED FILES..
  AfterInstall:
    - location scripts/webserver-start.sh
      timeout: 300
      runas: root

========================================================================================================================================
BELOW IS A VERSION OF "appspec.yml" THAT IS BUILT FOR LINUX IN THE GIVEN EXAMPLE

NEXT LINE IS LINE 1 OF APPSPEC FILE

version: 0.0
os: linux

files:
  - source: myfolder/myfile
    destination: /myfolder/

permissions:
  - object: /myfolder
    owner: ec2-user
    group: minedminds
    mode: 4755
    acls:
      - u:ec2-user:rwx
      - g:minedminds:rwx
    type:
      - directory

hooks:
  BeforeInstall:
    - location: scripts/webserver-stop.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/webserver-start.sh
      timeout: 300
      runas: root


