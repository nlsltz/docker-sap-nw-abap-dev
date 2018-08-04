# SAP NW ABAP Trial in Docker

Useful for setting up a local ABAP for own education. Not intended for production. After all, we're putting a fat monolith into Docker. However, using Docker still allows you to keep your host system clean of all the mess any installation can cause.

See Nabis YouTube video for additional details: [Installing SAP NW ABAP 7.51 SP02 into Docker](https://www.youtube.com/watch?v=H0GEg8r7P48)

Check also Nabis blog [Installing SAP NW ABAP into Docker](https://blogs.sap.com/2018/05/30/installing-sap-nw-abap-into-docker/). There you'll find links to the whole blog series.

## Attribution

The Dockerfile is based on:

- [This Dockerfile by Gregor Wolf](https://bitbucket.org/gregorwolf/dockernwabap750/src/25ca7d78266bef8ed41f1373801fd5e63e0b9552/Dockerfile?at=master&fileviewer=file-view-default)
- [This Dockerfile by Tobias Hofmann](https://github.com/tobiashofmann/sap-nw-abap-docker/blob/master/Dockerfile)

## Instructions

1.  Install [Docker](https://www.docker.com/community-edition)

1.  Increase the `Disc Image Size` in your Docker preferences

    - Just add 100 GB to the existing value :-)
    - You may want to increase the `Memory` in Docker's advanced settings (I chose `6 GiB`)

1.  Install [Git](https://git-scm.com)

    On Windows I suggest to install Git Bash as well (you'll be asked during the installation process).

    **Hint:** Installing git is actually not really needed. Alternatively, you could also copy/download this Dockerfile etc. to your machine.

1.  Clone this repo

    ```sh
    git clone https://github.com/nzamani/sap-nw-abap-trial-docker.git
    cd sap-nw-abap-trial-docker
    ```

1.  Download [SAP NW ABAP 7.51 Trial from SAP](https://tools.hana.ondemand.com/#abap) from [SAP Store](https://store.sap.com/sap/cp/ui/resources/store/html/SolutionDetails.html?pid=0000014493&catID=&pcntry=DE&sap-language=EN&_cp_id=id-1477346420741-0), then:

    - Create a folder `sapdownloads` inside the clone
      - `mkdir sapdownloads`
    - Extract the downloaded rar files into the folder we just created

    **Hint:** SAP wants to know who downloads the NW ABAP Trial version. Thus, you will have to logon with your own account before you can start the download. Creating an account is free, so is the download. The account can be the same account you use for the SAP Communitiy / SCN.

1.  Build the Docker image

    - Without Proxy

      ```sh
      docker build -t nwabap:7.51 .
      ```

    - Behind a Proxy

      ```sh
      docker build --build-arg http_proxy=http://proxy.mycompany.corp:1234 --build-arg https_proxy=http://proxy.mycompany.corp:1234 -t nwabap:7.51 .
      ```

      **Hint:** In a proxy environment your `docker build` command (see above) will fail in case you don't set the proxy as mentioned above or in case you use wrong proxy settings. Also consider that you might have to set the proxy manually for some software installed in the container.

1.  Create/Start a container with one of the following commands:

    - Use this if you want to map the default SAP ports as they come on localhost (preferred)

      ```sh
      docker run -p 8000:8000 -p 44300:44300 -p 50000:50000 -p 50001:50001 -p 3300:3300 -p 3200:3200 -h vhcalnplci --name nwabap751 -it nwabap:7.51 /bin/bash
      ```

    - Use this one if "random" ports on localhost are fine for you

      ```sh
      docker run -P -h vhcalnplci --name nwabap751 -it nwabap:7.51 /bin/bash
      ```

    **Hint:** You could also use `--rm` to make the container is removed after you exit your cli/terminal, i.e.:

    ```sh
    docker run -p 8000:8000 -p 44300:44300 -p 50000:50000 -p 50001:50001 -p 3300:3300 -p 3200:3200 -h vhcalnplci --rm --name nwabap751 -it nwabap:7.51 /bin/bash
    ```

1.  Now start the installation of SAP NW ABAP 7.51 Trial

    - Auto install via Expect script (suggested for simplicity)

      ```sh
      /usr/sbin/uuidd
      ./install.exp
      ```

    - Or the standard way

      ```sh
      /usr/sbin/uuidd
      ./install.sh
      ```

    Your installation has been successful if you see the followong message: **Installation of NPL successful**

    **Hint:** This installation will take about 20-30 minutes. Once done your SAP is running. Next, stop the system and exit the container.

## Starting and Stopping the NW ABAP 7.51 Trial

1.  Starting the container + SAP NW ABAP Trial (use this from now on instead of `docker run ...` from above)

    ```sh
    docker start -i nwabap751
    /usr/sbin/uuidd
    su npladm
    startsap ALL
    ```

1.  Stopping SAP NW ABAP Trial and container (`ALL` can be omitted)

    ```sh
    su npladm
    stopsap ALL
    exit
    exit
    ```

    **Hint:** After the second `exit` the Docker container is stopped.

## Important Post Installation Steps

1.  Updating License

    - Open SAP GUI and logon

      - **User:** SAP\*
      - **Password:** Appl1ance
      - **Client:** 000

    - Open Transaction `SLICENSE`
    - From the Screen copy the value of field `Active Hardware Key`
    - Go to [SAP License Keys for Preview, Evaluation, and Developer Versions](https://go.support.sap.com/minisap/#/minisap) in your browser
    - Choose `NPL - SAP NetWeaver 7.x (Sybase ASE)`
    - Fill out the fields. Use the `Hardware Key` you copied from `SLICENSE`
    - Keep the downloaded file `NPL.txt` and go back to the `SLICENSE`
    - Delete the `Installed License` from the table
    - Press the button `Install` below the table
    - Choose the downloaded file `NPL.txt`
    - Done - happy learning. Now logon with the dev user.

    You can now logon to `client 001` with any of the following users (all share the same password `Appl1ance`, typically you would work with `DEVELOPER`):

    - **User:** DEVELOPER (Developer User)
    - **User:** BWDEVELOPER (Developer User)
    - **User:** DDIC (Data Dictionary User)
    - **User:** SAP\* (SAP Administrator)

1.  Generating Test Data

    Execute the following to generate some test data:

    - **Report:** SAPBC_DATA_GENERATOR
    - **Transaction Code:** SEPM_DG

1.  Set up transaction favorites

    - Open SAP GUI
    - From menu "Favorites" -> "Upload from PC" and select `sap-gui-transaction-favorites.txt`
    - Choose to "append" the uploaded favorites

1.  Set up ABAP Development Tools for Eclipse

    - Go to Transaction `SICF`
    - Activate the node `/sap/bc/adt` (default_host)
    - Test the ADT connection with Eclipse

      - Open Eclipse Oxygen
      - Install ADT
      - Switch to ABAP perspective
      - Create new ABAP project

        - Select NPL system from list
        - Add system credentials
        - Hit `Finish` and see if connection succeeds

1.  Add hosts entry to access Fiori Launchpad from container host

    - **MacOS**

      - Open `/ect/hosts` as administrator
      - Add `127.0.0.1 vhcala4hci.wdf.sap.corp` at the end of hosts file
      - Save file and flush DNS cache with `sudo dscacheutil -flushcache`
      - One-liner `sudo -- sh -c 'echo "127.0.0.1 vhcala4hci.wdf.sap.corp" >> /etc/hosts; sudo dscacheutil -flushcache'`

        - **Fiori Launchpad:** [https://vhcala4hci.wdf.sap.corp:50001/sap/bc/ui5_ui5/ui2/ushell/shells/abap/FioriLaunchpad.html](https://vhcala4hci.wdf.sap.corp:50001/sap/bc/ui5_ui5/ui2/ushell/shells/abap/FioriLaunchpad.html)
        - **Launchpad Designer:** [https://vhcala4hci.wdf.sap.corp:50001/sap/bc/ui5_ui5/sap/arsrvc_upb_admn/main.html](https://vhcala4hci.wdf.sap.corp:50001/sap/bc/ui5_ui5/sap/arsrvc_upb_admn/main.html)

1.  Set up abapGit

    - Configure SSL

      - Add SSL parameters

        - Go to profile dir on OS level (`cdpro`)
        - Edit file `NPL_D00_vhcalnplci`
        - Add line `ssl/client_ciphersuite = 150:PFS:HIGH::EC_P256:EC_HIGH`
        - Add line `ssl/ciphersuite = 135:PFS:HIGH::EC_P256:EC_HIGH`
        - Save file
        - Restart system

      - Restart ICM process

        - Open transaction `SMICM`
        - From menu "Administration" -> "ICM" -> "Exit Soft" -> "Global"

    - Install GitHub SSL certificates

      - Open Chrome and go to https://github.com
      - Click the green locker near the address bar and hit "Show certificate"
      - Drag all certificates showing to your desktop
      - Open transaction `STRUST`
      - Switch to `CHANGE` mode
      - Double-click "SSL client SSL Client (Anonymous)" folder
      - In the third box, Certificate, click on "Import certificate", select the certificate from your desktop and click "Add to certificate list"
      - Save

    - Install abapGit

      - Create `ZABAPGIT` report in transaction `SE38`
      - Copy/paste source code from [zabapgit.abap](https://raw.githubusercontent.com/abapGit/build/master/zabapgit.abap) into report
      - Activate report
      - Run report
      - Click "install abapGit repo"

1.  Suggestion: Activate the good old ping service

    - Go to Transaction `SICF`
    - Activate the node `/sap/public/ping` (default_host)
    - Test the HTTP and HTTPS connection with your browser

      - **HTTP:** [https://vhcala4hci.wdf.sap.corp:8000/sap/public/ping](http://vhcala4hci.wdf.sap.corp:8000/sap/public/ping)
      - **HTTPS:** [https://vhcala4hci.wdf.sap.corp:44300/sap/public/ping](https://vhcala4hci.wdf.sap.corp:44300/sap/public/ping)

## Logfiles

Assuming you have started your container (not neccessaryly the NW ABAP in the container) and switched to user `npladm`:

```sh
docker start -i nwabap751
/usr/sbin/uuidd
su npladm
```

Afterwards, type `alias` to see some shortcuts SAP has created for us:

```sh
alias
```

One of them is `cdDi` which you can execute in you CLI:

```sh
cdDi
```

Then, after hitting `ls -ahl` you know there is a directory `work`:

```sh
cd work
```

This folder contains important log files for us, i.e. `dev_icm` and `dev_w0`:

```sh
vi dev_icm
vi dev_w0
```

Check their content (i.e. with vi) in case you're facing issues with your NW ABAP which you can't explain. For example, in case your NW ABAP has started successfully, but you cannot access the ping service via HTTP/HTTPS, you might find an issue in one of these files.
