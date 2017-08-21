# ClientCertificateMiddleware
Asp.net core Client Certificate Middleware
The package is targets netstandard2.0.

The Client Certificate Middleware will authorize a request based on the configured AuthorizedCertficatesAndRoles

Example:
```sh
  "AuthorizedCertficatesAndRoles": {
    "CertificateAndRoles": [
      {
        "Subject": "CN=localhost",
        "Issuer": "CN=localhost",
        "Roles": [ "User" ]
      },
      {
        "Subject": "CN=Store",
        "Issuer": "Me",
        "Roles": [ "User" ]
      }
    ]
  }
```

To run the demonstration, you need to install a certificate and gives its subject in the configuration.
```sh
"ServerCertificateSubject": "CN=localhost",
```

To create a certificate you can run PowerShell as admin and run
```sh
# Generate server certificate
$cert = New-SelfSignedCertificate -DnsName http://clientcertificatemiddlewaredemo.azurewebsites.net -CertStoreLocation "cert:\LocalMachine\My"
$password = ConvertTo-SecureString -String "your-password" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath "\temp\clientcertificatemiddlewaredemo.pfx" -Password $password

# Generate user certificates
$password = ConvertTo-SecureString -String "password" -Force -AsPlainText
$certUser = New-SelfSignedCertificate -DnsName http://user.mylocalmachine -CertStoreLocation "cert:\LocalMachine\My"
$certAdmin = New-SelfSignedCertificate -DnsName http://admin.mylocalmachine -CertStoreLocation "cert:\LocalMachine\My"
Export-PfxCertificate -Cert $certUser -FilePath "\temp\user.mylocalmachine.pfx" -Password $password
Export-PfxCertificate -Cert $certAdmin -FilePath "\temp\admin.mylocalmachine.pfx" -Password $password
```
On Dev machine, you have to install the certificate to Current User -> Trusted Root Certification Authorities
Otherwise you will see the exception "The remote certificate is invalid according to the validation procedure."

## Azure

### Import the SSL into Azure. 
Go to your Azure Web application
> SSL Certificates  
>> Upload Certificate.
>> Note the Thumbprint

> Application Setting
>> Add WEBSITE_LOAD_CERTIFICATES and the Thumbprint
>> Add ASPNETCORE_ENVIRONMENT  and setting like 'Staging'


To get the certificates to work, don't run the demo under IIS Express. Instead run under the app 'ClientCertificateMiddlewareDemo'
Refer here for more information.
http://www.blinkingcaret.com/2017/03/01/https-asp-net-core/


Here is the code that sets up the use of client certificate
```sh
.UseKestrel(options =>
{
    options.Listen(new IPEndPoint(IPAddress.Loopback, 4430), listenOptions =>
    {
        var httpsConnectionAdapterOptions = new HttpsConnectionAdapterOptions()
        {
            ClientCertificateMode = ClientCertificateMode.AllowCertificate,
            SslProtocols = System.Security.Authentication.SslProtocols.Tls,
            ServerCertificate = certificate
        };
        listenOptions.UseHttps(httpsConnectionAdapterOptions);
    });
})
```
