---
title: "TOTP 2FA in Golang Applications"
seoDescription: "Implement TOTP 2FA in Golang for better security with pquerna/otp, Twilio, or Firebase"
datePublished: Thu Oct 12 2023 07:00:10 GMT+0000 (Coordinated Universal Time)
cuid: clnmtyfbj000y09l01kyn4enw
slug: totp-2fa-in-golang-applications
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697022397839/41f6dedc-052a-4e45-8f7f-15811ada91f8.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1696846446673/7bbd8a0c-f73a-441f-9751-6246c6200439.png
tags: go, golang, passwords, one-time-password, two-factor-authentication-2fa

---

## Introduction

Time-Based One-Time Password (TOTP) is a widely used two-factor authentication (2FA) mechanism that enhances the security of online accounts and systems. TOTP generates temporary, one-time passwords that expire after a short period of time, typically 30 seconds. These passwords are generated based on a shared secret key and the current time, ensuring that they are dynamic and highly secure.

Here’s how TOTP works:

1. **Initial Setup**: During the initial setup of 2FA for an account, the user associates their account with a TOTP-compatible authentication app or device. This step typically involves scanning a QR code or manually entering a shared secret key provided by the service.
    
2. **Shared Secret Key**: A secret key is securely stored on both the server and the user’s device. This key is used as the basis for generating TOTP codes.
    
3. **Code Generation**: When the user attempts to log in or perform a sensitive action, they open the TOTP app, which calculates a time-based code using the shared secret key and the current time. This code is typically a 6- or 8-digit numerical value.
    
4. **Validation**: The user enters the generated TOTP code on the login page. The server also calculates the expected TOTP code based on the shared secret key and the current time window. If the user’s entered code matches the server’s calculated code, access is granted.
    
5. **Time-Based Security**: TOTP codes are time-sensitive, meaning they change every 30 seconds (or a configurable time interval). This time-based aspect adds an extra layer of security, as codes quickly become invalid.
    

TOTP is a secure and convenient method for implementing 2FA, as it doesn’t require a network connection during code generation, making it resistant to many common attack vectors. Popular authentication apps like Google Authenticator and Authy support TOTP, making it accessible to a wide range of users and services seeking to enhance their online security.

## Flow

The flow to authenticate with TOTP (this with Twilio service API but the flow is quite similar between the services) is demonstrated in the sequence diagram below:

![register-user-public-docs-sequence-diagram-Verify_TOTP_Sequence_Diagram 4.png](https://assets.cdn.prod.twilio.com/images/register-user-public-docs-sequence-diagram-Ver.width-800_5Wq873t.png align="left")

This diagram showcases the data flow to register new TOTP credential (TOTP seeding).

![verify-user-public-docs-sequence-diagram-Verify_TOTP_Sequence_Diagram 3.png](https://assets.cdn.prod.twilio.com/images/verify-user-public-docs-sequence-diagram-Verif.width-800_LpJ3zMz.png align="left")

This diagram shows how TOTP works when we login into the system (validate user).

The Time-based One-Time Password (TOTP) algorithm works under the hood by combining a shared secret, a timestamp, and a cryptographic hash function to generate a one-time password that changes over time. Here’s a summarized overview of how TOTP flows:

1. **Shared Secret**: TOTP relies on a shared secret, typically generated and securely stored by the server and shared with the user during the initial setup. This secret is a random value and must remain secret to ensure the security of the system.
    
2. **Timestamp**: TOTP uses a timestamp, often in the form of the current Unix time (seconds since the Unix epoch, which is January 1, 1970). This timestamp serves as a time reference point.
    
3. **Hash Function**: TOTP employs a cryptographic hash function, commonly HMAC-SHA1, HMAC-SHA256, or HMAC-SHA512. This function takes two inputs: the shared secret and the timestamp, and it produces a fixed-length hash value as output.
    
4. **Time Step**: TOTP defines a time step, which is typically set to 30 seconds. The timestamp is divided by the time step to determine the current time step.
    
5. **Counter**: The current time step, along with the shared secret, is used as inputs to the hash function. The result is a hash value.
    
6. **Dynamic Truncation**: The hash value is dynamically truncated to create a shorter value. The truncation involves selecting a portion of the hash, usually the lower bits, to create a smaller number.
    
7. **OTP Generation**: The truncated value is processed to create a one-time password (OTP). This is often done by converting the truncated value into a numerical representation, applying modulo operations to limit the length, and optionally adding leading zeros.
    
8. **OTP Display**: The OTP is displayed to the user through a trusted authentication application (such as Google Authenticator or Authy) or hardware token.
    
9. **Validation**: When the user attempts to log in, they enter the OTP generated by their authentication app. The server also calculates the expected OTP based on the shared secret and the current timestamp.
    
10. **Time Drift Tolerance**: To account for time synchronization issues and device clock drift, the server may check for OTP validity in the current and adjacent time steps.
    
11. **Validation Success**: If the OTP entered by the user matches the expected OTP calculated by the server, access is granted. Otherwise, access is denied.
    

In summary, TOTP generates one-time passwords by combining a secret key, a timestamp, and a cryptographic hash function. This process ensures that the **generated passwords change over time**, enhancing security by requiring a new password for each authentication attempt. Users and servers synchronize their clocks to ensure accurate OTP generation and validation, and time-based validity windows are used to account for potential time discrepancies. This combination of elements makes TOTP an effective and widely adopted method for two-factor authentication.

## Implementation in Go

We can use our way to implement TOTP with a nearly identical flow for verification with [`https://github.com/pquerna/otp`](https://github.com/pquerna/otp) library.

Here is a simple implementation of the library:

```go
package main

import (
	"github.com/pquerna/otp"
	"github.com/pquerna/otp/totp"

	"bufio"
	"bytes"
	"encoding/base32"
	"fmt"
	"image/png"
	"io/ioutil"
	"os"
	"time"
)

func display(key *otp.Key, data []byte) {
	fmt.Printf("Issuer:       %s\\n", key.Issuer())
	fmt.Printf("Account Name: %s\\n", key.AccountName())
	fmt.Printf("Secret:       %s\\n", key.Secret())
	fmt.Println("Writing PNG to qr-code.png....")
	ioutil.WriteFile("qr-code.png", data, 0644)
	fmt.Println("")
	fmt.Println("Please add your TOTP to your OTP Application now!")
	fmt.Println("")
}

func promptForPasscode() string {
	reader := bufio.NewReader(os.Stdin)
	fmt.Print("Enter Passcode: ")
	text, _ := reader.ReadString('\\n')
	return text
}

func main() {
	key, err := totp.Generate(totp.GenerateOpts{
		Issuer:      "Example.com",
		AccountName: "alice@example.com",
	})
	if err != nil {
		panic(err)
	}
	// Convert TOTP key into a PNG
	var buf bytes.Buffer
	img, err := key.Image(200, 200)
	if err != nil {
		panic(err)
	}
	png.Encode(&buf, img)

	// display the QR code to the user.
	display(key, buf.Bytes())

	// Now Validate that the user's successfully added the passcode.
	fmt.Println("Validating TOTP...")
	passcode := promptForPasscode()
	valid := totp.Validate(passcode, key.Secret())
	if valid {
		println("Valid passcode!")
		os.Exit(0)
	} else {
		println("Invalid passcode!")
		os.Exit(1)
	}
}
```

Look simple enough. Let’s break it down.

* The function `display` is for displaying and generating the QR code (in this case replacing the FE).
    
* The `promptForPasscode` function is for the user to input the passcode.
    
* in `main` function:
    
    * First, we generate a key ( this key is for registering the credential ) and this key holds the secret to the credential that must be saved in the DB ( replace the factor SID ).
        
    * Then we generate an image from the QR code of the key
        
    * Then display the QR code to the user.
        
    * The user then scans the QR code with their favorite authenticate app (MS Auth, Authy, Google Auth,…).
        
    * Then call `promptForPasscode` function to prompt a field for user to input the passcode in the app and verify it with `Validate` method.
        

As you can see the flow is quite similar for any other 3rd party services but with this, we have to manage everything ourselves.

If you run the code above it will be something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696846124883/f7d9c747-c151-4a59-bb93-18b0aca0a8e2.png align="center")

And your Directory should have `qr-code.png` image

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696846173225/bbd7a4a0-ff4f-4a4b-9027-dc5089278d69.png align="center")

Scan the QR code with your favorite Authenticator ( Google Authenticator, MS Authenticator, Twilio Authy,…) and input the passcode into the prompt you will get something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696846210584/b3b82176-7459-4fb1-b5fe-c3fe8f33c30b.png align="center")

## Alternative

We can also use other off-the-shelf services to save the effort of maintaining and upgrading

[https://www.twilio.com/docs/verify/quickstarts/totp](https://www.twilio.com/docs/verify/quickstarts/totp)

[https://firebase.google.com/docs/auth/web/totp-mfa](https://firebase.google.com/docs/auth/web/totp-mfa)

## Conclusion

The implementation of Two-Factor Authentication (2FA) using the Time-based One-Time Password (TOTP) algorithm in Go represents a significant advancement in cybersecurity. TOTP, with its reliance on time-based codes generated by a shared secret, adds an invaluable layer of security by requiring an additional one-time code, even if an attacker gains access to the user's password. The Go programming language's efficiency, simplicity, and a strong standard library make it a pragmatic choice for implementing TOTP-based 2FA, allowing developers to easily create TOTP generators and verifiers. Overall, the TOTP implementation in Go enhances security and demonstrates the adaptability of the language in addressing cybersecurity challenges.

## Reference:

* [https://www.twilio.com/docs/verify/totp/technical-overview](https://www.twilio.com/docs/verify/totp/technical-overview)
    
* [https://en.wikipedia.org/wiki/Time-based\_one-time\_password](https://en.wikipedia.org/wiki/Time-based_one-time_password)
    
* [https://github.com/pquerna/otp](https://github.com/pquerna/otp)
    

### ***Contributing***

*At Dwarves, we encourage our people to read, write, share what we learn with others, and* [***contributing to the Brainery***](https://brain.d.foundation/CONTRIBUTING) *is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.*

### *Love what we are doing?*

* *Check out our* [***products***](https://superbits.co/)
    
* *Hire us to* [***build your software***](https://d.foundation/)
    
* *Join us,* [***we are also hiring***](https://github.com/dwarvesf/WeAreHiring)
    
* *Visit our* [***Discord Learning Site***](https://discord.gg/dzNBpNTVEZ)
    
* *Visit our* [*GitHub*](https://github.com/dwarvesf)