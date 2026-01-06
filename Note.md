
* can we add one field called "is_new_user: boolean" for login response  

> openssl genrsa -out jwt_private.pem 2048
> awk '{printf "%s\\n", $0}' jwt_private.pem





* not to expose sensitive information in /login api response - Done
* keep secret key approach instead of private key - Done
* not to keep encrypted password in db due to security concern, instead do some hashes or some logic to get password dynamically same

- Email and Phone Number make one as mandatory field




**My Note:**
- make email/mobile is mandatory, currently we have mandatory in keycloak check on that and make all is of condition in json schema (either mobile or phone number)


Register(ctx context.Context, req *request.RegistrationRequest) (*models.Account, string, error) -> why string



remove ACCOUNT_PASSWORD_POLICY variable






Note:
	2025.12.23.GA tag





Approach

to make changes in keycloak token we need to have keycloak encryption key(we get it from keycloak database) so, when we have that key, we can modify the token claims and again we can sign for that token

