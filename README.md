# Mallard Theme

## FTP

We use the [Basic FTP](https://www.npmjs.com/package/basic-ftp) library to upload and dowload files.

You will need to create a `.env` file with the FTP credentials.

Use this format:

```.env
# FTP Environment - deploying to live or a dev environment
# Values should be "live" for the production environment or "dev" for
# sites that are run in the Aptuitiv Developer Development environment for testing.
FTP_ENVIRONMENT = live

# Live server FTP information
FTP_SERVER = ftp1.branchcms.com
FTP_USERNAME = my_ftp_username
FTP_PASSWORD = my_ftp_password

# Development server FTP information
FTP_DEV_SERVER = TBD
FTP_DEV_USERNAME = TBD
FTP_DEV_PASSWORD = TBD
```

The `FTP_ENVIRONMENT` should be set to `live` for uploading to production sites. Set the following variables:

-   `FTP_SERVER`: The FTP server
-   `FTP_USERNAME`: The FTP username
-   `FTP_PASSWORD`: The FTP password

If you are using a clone of the website in the Aptuitiv Development environment for testing then set `FTP_ENVIRONMENT` to "dev" and set the `FTP_DEV_SERVER`, `FTP_DEV_USERNAME`, and `FTP_DEV_PASSWORD`.
