# Time-based One Time Password (MFA) Library for Java

[![CircleCI](https://circleci.com/gh/samdjstevens/java-totp/tree/master.svg?style=svg&circle-token=10b865d8ba6091caba7a73a5a2295bd642ab79d5)](https://circleci.com/gh/samdjstevens/java-totp/tree/master) [![Maven Central](https://img.shields.io/maven-central/v/dev.samstevens.totp/totp.svg?label=Maven%20Central)](https://search.maven.org/search?q=g:%22dev.samstevens.totp%22%20AND%20a:%22totp%22)

A java library to help generate and verify time-based one time passwords for Multi-Factor Authentication.

Generates QR codes that are recognisable by applications like Google Authenticator, and verify the one time passwords they produce.

Inspired by [PHP library for Two Factor Authentication](https://github.com/RobThree/TwoFactorAuth), a similar library for PHP.

## Requirements

- Java 8+



## Installation

#### Maven

To add this library to your java project using Maven, add the following dependency:

```xml
<dependency>
  <groupId>dev.samstevens.totp</groupId>
  <artifactId>totp</artifactId>
  <version>1.2</version>
</dependency>
```

#### Gradle

To add the dependency using Gradle, add the following to the build script:

```
dependencies {
  compile 'dev.samstevens.totp:totp:1.2'
}
```



## Usage

### Generating a shared secret

To generate a secret, use the `dev.samstevens.totp.secret.DefaultSecretGenerator` class.
```java
SecretGenerator secretGenerator = new DefaultSecretGenerator();
String secret = secretGenerator.generate();
// secret = "BP26TDZUZ5SVPZJRIHCAUVREO5EWMHHV"
```

By default, this class generates secrets that are 32 characters long, but this number is configurable via

the class constructor.

```java
// Generates secrets that are 64 characters long
SecretGenerator secretGenerator = new DefaultSecretGenerator(64);
```



### Generating a QR code

Once a shared secret has been generated, this must be given to the user so they can add it to an MFA application, such as Google Authenticator. Whilst they could just enter the secret manually, a much better and more common option is to generate a QR code containing the secret (and other information), which can then be scanned by the application.

To generate such a QR code, first create a `dev.samstevens.totp.qr.QrData` instance with the relevant information.

```java
 QrData data = new QrData.Builder()
   .label("example@example.com")
   .secret(secret)
   .issuer("AppName")
   .digits(6)
   .period(30)
   .build();
```

Once you have a `QrData` object holding the relevant details, a PNG image of the code can be generated using the `dev.samstevens.totp.qrZxingPngQrGenerator` class.

```java
QrGenerator generator = new ZxingPngQrGenerator();
byte[] imageData = generator.generate(data)
```

The `generate` method returns a byte array of the raw image data. The mime type of the data that is generated by the generator can be retrieved using the `getImageMimeType` method.

```java
String mimeType = generator.getImageMimeType();
// mimeType = "image/png"
```

The image data can then be outputted to the browser, or saved to a temporary file to show it to the user.



### Verifying one time passwords

After a user sets up their MFA, it's a good idea to get them to enter two of the codes generated by their app to verify the setup was succerssful. To verify a code submitted by the user, do the following:

```java
TimeProvider timeProvider = new SystemTimeProvider();
CodeGenerator codeGenerator = new DefaultCodeGenerator();
CodeVerifier verifier = new DefaultCodeVerifier(codeGenerator, timeProvider);

// secret = the shared secret for the user
// code = the code submitted by the user
boolean successful = verifier.isValidCode(secret, code)
```

This same process is used when verifying the submitted code every time the user needs to in the future.



## Running Tests

To run the tests for the library with Maven, run `mvn test`.




## License

This project is licensed under the [MIT license](https://opensource.org/licenses/MIT).
