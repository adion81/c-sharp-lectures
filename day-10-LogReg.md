# Day 10 - LoginReg

<img src="https://miro.medium.com/max/480/1*SnZqHENpIMiEKsg999Q0DQ.png" alt="ef-core" />

## LoginReg

In order to have a successful Login & Registration we need to validate both a Register Form and a Login Form.<br>
<br>
We do so by creating two different models: `User.cs` && `LoginUser.cs`

#### User.cs
This model will get put into our database.

```cs
    namespace LoginReg.Models
    {
        public class User
        {
            [Key]
            public int UserId { get; set; }

            [Required]
            public string FirstName {get;set;}

            [Required]
            public string LastName {get;set;}

            [EmailAddress]
            [Required]
            public string Email {get;set;}

            [DataType(DataType.Password)]
            [Required]
            [MinLength(8, ErrorMessage="Password must be 8 characters or longer!")]
            public string Password {get;set;}

            public DateTime CreatedAt {get;set;} = DateTime.Now;
            public DateTime UpdatedAt {get;set;} = DateTime.Now;

            // We use the NotMapped Annotation so that this variable doesn't end up in our database.
            [NotMapped]
            [Compare("Password")]
            [DataType(DataType.Password)]
            public string Confirm {get;set;}
        }
    }
```
#### LoginUser.cs
This model will not get put into our database.

```cs

    namespace LoginReg.Models
    {
        public class LoginUser
        {
            [Required]
            [EmailAddress]
            public string LoginEmail { get; set; }

            [DataType(DataType.Password)]
            [Required]
            public string LoginPassword {get;set;}
        }
    }
```

### Hashing Passwords

In order to hash a password we must import Identity from AspNetCore.
`using Microsoft.AspNetCore.Identity;`

Then we can hash passwords like so:

#### Somewhere in your HomeController.cs

```cs
    [HttpPost("register")]
    public IActionResult Register(User reg)
    {
        //These two lines will hash our password for us.
        PasswordHasher<User> Hasher = new PasswordHasher<User>();
        reg.Password = Hasher.HashPassword(reg, reg.Password);
    }

    [HttpPost("login")]
    publc IActionResult Login(LoginUser log)
    {

        //These two lines will compare our hashed passwords.
        var hash = new PasswordHasher<LoginUser>();
        var result = hash.VerifyHashedPassword(log, userInDb.Password, log.LoginPassword);
        //Result will either be 0 or 1.
        if(result == 0)
        {
            //Handle rejection and send them back to the form.
        } 

    }
```
### Logic steps for Registration
```
    1. If the User model is valid.
        3. If the email coming in is not in the database.
            5. Hash the password.
            6. Save the hashed version to the user model.
            7. Add it to the database.
            8. Save the database.
            9. Put the UserId into Session.
            10. Redirect!!!
        4. Else update the validations and render the view with the Registrtion Form.
    2. Else render the view with the Registration Form.
```

### Logic steps for Login
```
    1. If the LoginUser model is valid.
        3. Query the database for the User based on the email.
        4. If the user is in the database.
            6. If the hashed password from the database matches the password coming in.
                8. Put the UserId into Session.
                9. Redirect!!!
            7. Else update the validatiohs and render the view with the Login Form.
        5. Else update the validations and render the view with the Login Form.
    2. Else render the view with the Login Form.
```