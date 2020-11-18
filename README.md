# Frontend

## Template
We have used the package.json and .eslintrc.js files found in the template from the DIT341 Web and Mobile Development course. 

The public mirror can be found [here](https://github.com/dit341/group-00-web).

## Getting started

### Mosquitto

1. Install [Mosquitto](https://mosquitto.org/)

2. Check that Users have the permissions to Modify/Write the mosquitto.conf file in the mosquitto folder.
   1. Right click on the mosquitto.conf file
   2. Choose Properties
   3. Navigate to the Security tab
   4. Click on Users, click Edit to change Permissions
   5. Select 'Modify' and 'Write'

3. Change the mosquitto.conf file by pasting in the below text into the top of the configuration file.

    ```port 1883
    listener 9001
    protocol websockets
    log_type all

4. To run Mosquitto with the configuration file in the Windows Command Prompt

    `mosquitto -c mosquitto.conf`

### Frontend

1. Clone the repository

    `git clone git@git.chalmers.se:courses/dit355/2020/group-3/frontend.git`

2. Change into the frontend directory

    `cd frontend`

3. Install dependencies

    `npm install`

4. Run the frontend server
 
    `npm run serve`
