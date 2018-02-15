# AAD B2C Identity Experience Framework - get the Facebook OAuth token
## Background
We all know that you can fairly easy [integrate Facebook authentication using Azure AD B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-setup-fb-app).
However, there are use cases where we not only want to authenticate the user, but also be able to talk to the [Facebook graph API](https://developers.facebook.com/docs/graph-api/).
That case is not covered by the standard policies and we have to use a custom policy.

> Everything in this tutorial assumes you are already familiar with Azure AD B2C custom policies and want more advanced stuff!
> I am not focusing on basics and only showing what you have to do differently.
> For tutorials on Custom Policies please follow the [official documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-overview-custom).

## Include a custom claim
In order to get anything down to our application, we have to define it as a `claim`. Here we will simply create a new claim in our ClaimsSchema.
So open your `TrustFrameworkExtensions.xml` file, locate `ClaimsSchema` under `BuildingBlocks` element, and add a new claim:

```xml
<BuildingBlocks>
    <ClaimsSchema>
      <ClaimType Id="identityProviderToken">
        <DisplayName>Identity Provider Token</DisplayName>
        <DataType>string</DataType>
        <UserHelpText />
      </ClaimType>
    </ClaimsSchema>
  </BuildingBlocks>
```

## Get the Facebook OAuth token from the Facebook exchange response
To populate our custom claim, we have to get its value. The original Facebook token can be found in the authentication response from Facebook.
Locate your `ClaimsProvider`, again in `TrustFrameworkExtensions.xml` and add one `OutputClaim` like this:

```xml
    <ClaimsProvider>
      <Domain>facebook</Domain>
      <DisplayName>Facebook</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Facebook-OAUTH">
          <Metadata>
            <Item Key="client_id">XXXXXXXXXX</Item>
            <Item Key="scope">email public_profile</Item>
            <Item Key="ClaimsEndpoint">https://graph.facebook.com/me?fields=id,first_name,last_name,name,email</Item>
          </Metadata>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="identityProviderToken" PartnerClaimType="{oauth2:access_token}" />
          </OutputClaims>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
``` 

You just need to refenrece the `access_token` *claim* from the Facebook *OAuth2* token like this `{oauth2:access_token}`.

## Gluing up
Last, but not least, you have to provide that new claim (`IdentityProviderToken`) to your application.
You do this in your Relaying Party policy. So locate your `LocalAndSocialAccount.xml` file (or however you have named it - 
this is basically your sign-up or sign-in policy file). And make appropriate changes to the UserJourney to return the 
`IdentityProviderToken` claim:

```xml
  <RelyingParty>
    <DefaultUserJourney ReferenceId="YOUR_DEFAULT_USER_JOURNEY_ID" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="identityProviderToken" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
```

You see the last `OutputClaim` in `OutputClaims` is the new `identityaProviderToken` claim, which we created. Now your application
also has access to the Facebook token and can call Facebook graph.

A complete [TrustFrameworkExtensions.xml](./TrustFrameworkExtensions.xml) file is located [here in this repository](./TrustFrameworkExtensions.xml). 
A [Relying Party File is also placed here](./SuSiLocalFb.xml) for your convenience.
Be aware - here I am only showing you what you need
to change, not how you can add Facebook as Identity Provider in a B2C tenant.
