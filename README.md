# SecureNotes (SSDExam)

Environment Setup

1. Clone the Repository

Terminal:
git clone https://github.com/ronnilenvighansen/SecureNotes.git

cd SecureNotes

2. Configure Environment Variables

Create a .env file in the project root with the following:

CONNECTION_STRING=Data Source=your-sqlite-db

Jwt_Authority=your-keycloak-url/realms/your-realm

Jwt_Audience=your-keycloak-client

CertPassword=your-cert-password

Cert=your-cert-folder/your-.pfx

3. Generate HTTPS Certificates

Create a folder:

mkdir certs

cd certs

Create localhost.conf with:

[req]

default_bits       = 2048

prompt             = no

default_md         = sha256

distinguished_name = dn

req_extensions     = req_ext

x509_extensions    = v3_req


[dn]

C  = DK

ST = Denmark

L  = Copenhagen

O  = MyCompany

OU = MyUnit

CN = localhost

[req_ext]

subjectAltName = @alt_names

[v3_req]

subjectAltName = @alt_names

[alt_names]

DNS.1 = localhost

Then generate your certs:

openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout your-.key \
  -out your-.crt \
  -config your-.conf

openssl pkcs12 -export \
  -out your-.pfx \
  -inkey your-.key \
  -in your-.crt \
  -passout pass:your-cert-password

4. Database Setup:

Install EF CLI if not already installed:

dotnet tool install --global dotnet-ef

Then initialize the SQLite database:

dotnet ef migrations add InitialCreate

dotnet ef database update

Running the Application:

dotnet run

Running Keycloak in Docker:

docker run -p 8443:8443 \
  -v "$PWD/your-cert-folder/your-.pfx:/etc/x509/https/keystore.pfx" \
  -v "$PWD/your-data-for-keycloak-folder:/opt/keycloak/data" \
  -v "$PWD/keycloak.conf:/opt/keycloak/conf/keycloak.conf" \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  -e KC_HTTPS_KEY_STORE_FILE=/etc/x509/https/keystore.pfx \
  -e KC_HTTPS_KEY_STORE_PASSWORD=your-cert-password \
  quay.io/keycloak/keycloak:latest \
  start --https-port=8443

5. Keycloak Configuration:

Realm:

Name: your-realm

Roles:

viewer

writer

deleter

Client:

Name: your-client

Root URL: your-https-launch-url

Redirect URIs: your-https-launch-url/signin-oidc

Web Origins: your-https-launch-url

Admin URL: your-https-launch-url

Client authentication: Enabled

User:

Username: your-user

Password: your-user-password

Roles: viewer, writer

Postman Setup:

Auth Type: OAuth 2.0

Grant Type: Authorization Code with PKCE

Auth URL:

your-https-launch-url/realms/your-realm/protocol/openid-connect/auth

Token URL:
your-https-launch-url/realms/your-realm/protocol/openid-connect/token

Client ID: your-client

Scope: openid

Code Challenge Method: SHA-256

Header Prefix: Bearer

6. API Endpoints:

Get Notes (for authenticated user):

GET your-https-launch-url/your-controller

Create Note:

POST your-https-launch-url/your-controller

Body (raw JSON):
{
  "content": "Hello!"
}

Delete All Notes:

DELETE your-https-launch-url/your-controller/your-api-endpoint

7. Notes:

This project uses HTTPS with a self-signed certificate.

Ensure your browser and Postman trust localhost.crt.

8. SBOM:

Install CycloneDX: dotnet tool install --global CycloneDX

Generate SBOM: 

dotnet CycloneDX ./SecureNotes.sln -j