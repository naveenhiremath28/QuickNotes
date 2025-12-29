
* can we add one field called "is_new_user: boolean" for login response  

> openssl genrsa -out jwt_private.pem 2048
> awk '{printf "%s\\n", $0}' jwt_private.pem





* dont expose sensitive information in /login api response
* keep secret key approach instead of private key
* dont keep encrypted password in db due to security concern, instead do some hashes or some logic to get password dynamically same