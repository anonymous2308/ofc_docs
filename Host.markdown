**Hosting Django in IIS**

Hosting Django Applications require certain pre-requisites:

1.  Python (latest)
2.  IIS
3.  Virtual Machine (for hosting)
4.  Postgres (DBMS)

Deployment of Django on IIS is done using **HttpPlatform handler** and the following steps are to be followed:\
**Note: All these to be followed in the virtual machine**

1.  **Preparation of project directory and python virtual environment:**\
    ![Project Directory](https://vestas-my.sharepoint.com/:i:/p/sjasb/ETmy4rG75EhLsZT26z1lh0EBFEHqa6VhqcROxTEgnVhvDQ?e=WURPYw)
    ![requirements.txt](https://vestas-my.sharepoint.com/:i:/p/sjasb/Ee50m2Gd41JPrHgeFjMRhDgBspunwetSIPAIyUjsdr_Njg?e=4u0rKG)

    a. This is the project directory inside which the python virtual environment is installed.
    b. Once virtual environment is installed, move inside the project directory from cmd window and activate it using **venv\Scripts\activate or call venv\Scripts\activate.bat**
    c. There is a '**requirements.**txt' text file that has to be maintained which contains all the required libraries to be installed, use **python -m pip install -r requirements.txt** execute the command

2.  **Installation of IIS:**

    a. Installation of IIS has to be using **Install-WindowsFeature -name Web-Server-IncludeManagementTools**
    b. By default , Handler Mappings comes with read access only, we need to change this by going into **IIS Manager > Server Level > Feature Delegation > Handler Mappings > Read/Write**
    ![Handler Mapping Access](https://vestas-my.sharepoint.com/:i:/p/sjasb/EQpMNLslNzZHvJShIxUdOGgBMzxArjreICTFWpLJEGP5mg?e=7DZdCs)

3.  **Deployment process:**

    a. **Open IIS, and create a new website**\

        i.  Provide a name for the site
        ii. Physical path: specify the **project directory
        iii. Change binding type from **http to https** and specify a port number
        iv. Select **incheanq01.vestas.net** SSL certificate and click **OK**

    ![New website](https://vestas-my.sharepoint.com/:i:/p/sjasb/EcpIuMycVPJPmEb_tlvQQ0wBBeC6ckMeZsDQgsjR6ev0Tg?e=j3b6v2)

    b. **Configure httpPlatform**,

    i. Go to **IIS Manager > Sites > Site01 > Configuration Editor > system.webServer/httpPlatform**

            1.  Arguments: path where "manage.py" is. Run it with "manage.py runserver %HTTP_PLATFORM_PORT%"
            2.  environmentVariables: an environment variable that will dynamically pass the value of the server variable "SERVER_PORT" into. %HTTP_PLATFORM_PORT%
            3.  processPath: path where "python.exe" is.
            4.  stdoutLogEnabled: True
            5.  stdoutLogFile: path where logs will be stored.

    ![**httpPlatform settings**](https://vestas-my.sharepoint.com/:i:/p/sjasb/EWIi0DHe9ylAqQvKKRXbtb8BNxnVRSoCrRlhtOGNOkfwNQ?e=q2HqEg)

    ii. Go to IIS manager \> Sites \> Sites01 \> Configuration Editor \> appSettings

            1.  PYTHONPATH: path to where you app is. In my case is pointing to mysite\<OUTER\>
            2.  WSGI_HANDLER: django.core.wsgi.get_wsgi_application()
            3.  DJANGO_SETTINGS_MODULE: path where "settings.py" is. Thanks to PYTHONPATH we just need to access "mysite\<INNER\>.settings". Hence value is: "mysite.settings"

    ![appSettings](https://github.com/anonymous2308/ofc_docs/blob/hostings/assets/appSettings.png)

    iii. Add module mapping, by going to **IIS manager \> Sites \> Sites01 \> Handler Mappings \> Add Module Mapping**

            1.  Request path: *
            2.  Module: httpPlatformHandler

    ![Module Mapping](https://vestas-my.sharepoint.com/:i:/p/sjasb/EYK8JZREtXFAqglTMCTUWwEBm5NaPEeJzQIUncjteEEh9w?e=XNOg1R)

    Then, click "Request Restrictions" and unchecked "Invoked handler only if requests is mapped to"\

    ![Request Restrictions](https://vestas-my.sharepoint.com/:i:/p/sjasb/EZw3fJjQ311Ll-3ID6HVzFIBs60j7WP_jA9jHMLc9LzWnQ?e=0DeCYZ)

    Click Ok and **web.config** should created in the project directory.

    c. **Editing web.config**\
     **Open web.config and replace it with the following**

        <?xml version="1.0" encoding="UTF-8"?>
        <configuration>
            <system.webServer>
                <httpPlatform processPath="change_project_directory\.venv\Scripts\python.exe" arguments="change_project_directory\manage.py runserver %HTTP_PLATFORM_PORT%" stdoutLogEnabled="true" stdoutLogFile="change_project_directory\logs">
                    <environmentVariables>
                        <environmentVariable name="SERVER_PORT" value="%HTTP_PLATFORM_PORT%" lockItem="false" />
                        <environmentVariable name="HTTPS" value="on"/>
                    </environmentVariables>
                </httpPlatform>
                <handlers>
                    <add name="prod-wms" path="*" verb="*" modules="httpPlatformHandler" resourceType="Unspecified" />
                </handlers>
                <!-- <httpRedirect enabled="false" destination="https://incheanq01.vestas.net:change_port_number/admin" exactDestination="true" /> -->
                <httpProtocol>
                    <customHeaders>
                        <add name="Access-Control-Allow-Origin" value="https://incheanq01.vestas.net" />
                        <add name="Access-Control-Allow-Credentials" value="true" />
                        <add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
                        <add name="Access-Control-Allow-Headers" value="Content-Type" />
                        <!-- <add name="Access-Control-Max-Age" value="86400" />
                        <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains"/>
                        <remove name="X-Powered-By"/> -->
                    </customHeaders>
                </httpProtocol>
                <!-- <rewrite>
                    <rules>
                        <rule name="HTTP to HTTPS redirect" stopProcessing="true">
                            <match url="(.*)" />
                            <conditions>
                                <add input="{HTTPS}" pattern="off" ignoreCase="true" />
                            </conditions>
                            <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
                        </rule>
                    </rules>
                </rewrite> -->
            </system.webServer>
            <appSettings>
                <add key="PYTHONPATH" value="change_project_directory" />
                <add key="WSGI_HANDLER" value="django.core.wsgi.get_wsgi_application()" />
                <add key="DJANGO_SETTINGS_MODULE" value="ServerEndApp.settings" />
            </appSettings>
        </configuration>

    **Change the project directory and port numbers accordingly**

    d. **Editing settings.py**

    i. **DEBUG = False**
    ii. **ALLOWED_HOSTS =[\'localhost\',\'10.211.97.85\',\'incheanq01.vestas.net\',\'127.0.0.1\'\]**
    iii. Make sure the order of MIDDLEWARE is followed as below

            MIDDLEWARE = [
                'corsheaders.middleware.CorsMiddleware',
                'django.middleware.security.SecurityMiddleware',
                'whitenoise.middleware.WhiteNoiseMiddleware', #add whitenoise
                'django.contrib.sessions.middleware.SessionMiddleware',
                'django.middleware.common.CommonMiddleware',
                'django.middleware.csrf.CsrfViewMiddleware',
                'django.contrib.auth.middleware.AuthenticationMiddleware',
                'django.contrib.messages.middleware.MessageMiddleware',
                'django.middleware.clickjacking.XFrameOptionsMiddleware',
                ]

    iv. Configure Postgres database as below\

            DATABASES = {
                'default': {
                    'ENGINE': 'django.db.backends.postgresql', # Use PostgreSQL as the database backend
                    'NAME': os.environ['DB_NAME_USAGE_LOGS'],  # database name
                    'USER': os.environ['DB_RTMS_BACKEND_CONN_USER'],
                    'PASSWORD': os.environ['DB_RTMS_BACKEND_CONN_PWD'],
                    'HOST': os.environ['DB_RTMS_HOST'],  # Database host, usually 'localhost'
                    'PORT': os.environ['DB_RTMS_PORT'],  # PostgreSQL port
                }
            }

    v. Include static files for visible css, js, images as below\

                STATIC_URL = 'static/'
                STATIC_ROOT = 'change_project_directory/static' ##specify static root

                STATICFILES_FINDERS = [
                    'django.contrib.staticfiles.finders.FileSystemFinder',
                    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
                ]

    also extract the static files from the project using the command **python manage.py collectstatic**

    vi. Change CORS_ALLOWED_ORGINS as below and specify the port numbers accordingly\

                CORS_ALLOWED_ORIGINS = [
                    # Add other allowed origins here
                    "https://incheanq01.vestas.net:7001",
                    "http://localhost:5000",
                ]

**Once all the above steps have been completed, restart the website in
IIS and launch the website and your good to go**
