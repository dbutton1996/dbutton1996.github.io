**Artifact Two**

**(Demonstrates Algorithms and Data Structures)**

For this artifact, I took the Authentication System that I created in Java for my IT 145: Foundation in Application Development course and ported it to Python while updating
the data structures to be more effective and secure. The purpose of this program is to prompt the user for a username and password, and then display a specific line of text depending on the credentials entered. Additionally, if invalid credentials are entered three times in a row, then the program will exit for security purposes. As one of my first Java programs, the original used a poorly designed method for storing and retrieving user data. Usernames, passwords, password hashes, and user role messages were stored in individual arrays, where the first entry in the username array corresponded to the first entry in the passwords array and so on. Additionally, passwords were stored in plaintext as well as MD5 hashes, making the hashes pointless as if the data was stolen the attackers could simply read the plaintext passwords.

**Original Code**

AuthenticationSystem.java
```java
package authentication.system;

import java.security.MessageDigest;
import java.util.Scanner;

public class AuthenticationSystem {
    
    public static String hasher(String userPassword) throws Exception{ //Encrypts user's password with MD5 encryption
        String original = userPassword;
        MessageDigest md = MessageDigest.getInstance("MD5");
        md.update(original.getBytes());
        byte[] digest = md.digest();
        StringBuffer sb = new StringBuffer();
        for (byte b : digest) {
            sb.append(String.format("%02x", b & 0xff));
        }

        return sb.toString();
    }

    public static void main(String[] args) throws Exception {
        String username = "";
        String password = "";
        String hashedPassword = new String();
        String logout = "";
        int totalTries = 0;
        int allowedTries = 3;
        int passwordLocation = -1; //Finds where the appropiate data is in the arrays used to store private data
        boolean accessGranted = false;
        boolean validInfo = false;
        boolean looping = true;
        credentials cred = new credentials();
        Scanner scnr = new Scanner(System.in);
        
        while (looping) { //If user logs out after successful login, ask them to log in again
            totalTries = 0;
            accessGranted = false;
            logout = "";

            while (totalTries < allowedTries) {
                System.out.print("Enter your username (Enter \"exit\" to exit): ");
                username = scnr.nextLine();
                if(username.equals("exit")) { //If the user enters "exit" for their username, close the program
                    return;
                }
                
                System.out.print("Enter your password: ");
                password = scnr.nextLine();
                hashedPassword = hasher(password); //Encrypts password
                validInfo = cred.checkHashedInfo(username, hashedPassword); //Sets validInfo to true if username and password both match an avaliable set
            
                totalTries += 1;
            
                if(validInfo) {
                    accessGranted = true;
                    break;
                }
                else {
                    System.out.println("Invalid credentials.");
                    System.out.println("Tries remaining: " + (allowedTries - totalTries) + "\n");
                }
            }
        
            if (totalTries > 3) { //Exit program if user fails to log in too many times
                System.out.println("Too many login attempts. Exiting program.");
                return;
            }
        
            while (accessGranted == true && !logout.equals("logout")) { //If user enters a valid username and password, display their role file
                System.out.println("");
                passwordLocation = cred.checkNumHashedPassword(hashedPassword); //Used to display correct role file
                cred.printUserRoles(passwordLocation); //Display role file
            
                while(!logout.equals("logout")) { //Keep asking user to type logout until they do
                    System.out.println("Type \"logout\" to logout.");
                    logout = scnr.nextLine();
                }
                System.out.println("");
            }
        }
        return;
    }
}
```
credentials.java
```java
package authentication.system;

public class credentials {
    final private String[] usernames = {"griffin.keyes", "rosario.dawson", "bernie.gorilla", "donald.monkey", "jerome.grizzlybear", "bruce.grizzlybear"};
    final private String[] hashes = {"108de81c31bf9c622f76876b74e9285f", "3e34baa4ee2ff767af8c120a496742b5", "a584efafa8f9ea7fe5cf18442f32b07b", "17b1b7d8a706696ed220bc414f729ad3", "3adea92111e6307f8f2aae4721e77900", "0d107d09f5bbe40cade3de5c71e9e9b7"};
    final private String[] passwords = {"alphabet soup", "animal doctor", "secret password", "M0nk3y business", "grizzly1234", "letmein"};
    final private String[] roles = {"zookeeper", "admin", "veterinarian", "zookeeper", "veterinarian", "admin"};
    
    public boolean checkHashedInfo(String username, String hashedPassword) {
        boolean isValidInfo = false;
        
        for(int i = 0; i < passwords.length; ++i ){ //Checks if the inputted hashed password matches one in the list
            if(hashedPassword.equals(hashes[i]) && username.equals(usernames[i])) {
                isValidInfo = true;
            }
        }
        
        return isValidInfo;
    }
    
    public int checkNumHashedPassword(String hashedPassword) { //Find and return's user's location in the arrays
        for(int i = 0; i < passwords.length; ++i) {
            if(hashedPassword.equals(hashes[i])) {
                return i;
            }
        }
        return -1;
    }
    
    public void printUserRoles(int role) { //Output role files
        String finalRole = roles[role];
        
        switch (finalRole) {
            case "admin":
                System.out.println("Hello, System Admin!\nAs administrator, you have access to the zoo\'s main computer system. This allows" + 
                                   " you to monitor users in the system and their roles.");
                break;
            
            case "zookeeper":
                System.out.println("Hello, Zookeeper!\nAs zookeeper, you have access to all of the animals\' information and their daily" + 
                                   " monitoring logs. This allows\nyou to track their feeding habits, habitat conditions, and general welfare.");
                break;
                
            case "veterinarian":
                System.out.println("Hello, Veterinarian!\nAs veterinarian, you have access to all of the animals\' health records. This allows" + 
                                   " you to view each animal\'s\nmedical history and current treatments/illnesses (if any), and to maintain" + 
                                   " a vaccination log.");
                break;
                
            default:
                System.out.println("Error, role not found");
        }
    }
}
```

When porting this program to Python, I completely overhauled the data structures used to store user data. Rather than storing data in arrays, it is instead stored within Python dictionaries, where each dictionary contains the data for a single user. This makes it easy to add, modify, or remove users from the system, as only Credentials.txt needs to be modified and not the Python program itself. Additionally, the securty used to protect the user's passwords has been greatly improved. Passwords are stored as a SHA-256 hash with two password salts included, one unique to each user and one globally to every user. This means if an attacker were to steal Credentials.txt, they would be unable to use the passwords in the file as they wouldn't know the user's password, the user's individual password salt, and the global password salt, all three of which are needed to create the hashed password.

**Updated Code**

AuthenticationSystem.py
```python
import hashlib
import ast

def loadCredentials(fileName): #Reads from .txt file and puts file's data in a list
    try:
        inputFile = open(fileName, "r")
        lines = inputFile.readlines()
        
        objects = []
        for line in lines:
            objects.append(ast.literal_eval(line))
        return objects
    except IOError:
        print("File not found. Contact system administrator.")
        exit()

username = ""
hashedPassword = ""
logout = ""
currentRole = ""
globalPasswordSalt = "BeaGGSXg^HzaqoxD4Ubix!jEbwieo7jK@dTndU!F^JH3efE*HLH45DU%rE97LUZMBSP$kAo!34$SJSEKMs*8ZxveNk7#zcXJ%d*ydee@H9QHqNge$T&3ApzioQhcnm%h" #Global password salt, added to each user's password
individualPasswordSalt = "" #Individual password salt, unique for each user
totalTries = 0
allowedTries = 3
validInfo = False

credentialsList = loadCredentials("Credentials.txt") #Load valid credentials from "Credentials.txt"
rollsList = loadCredentials("Roles.txt") #Load role messages from "Roles.txt"
    
while totalTries < allowedTries:
    logout = ""
    currentRole = ""
    validInfo = False
        
    username = input("Enter your username (Enter \"exit\" to exit): ")
    if username == "exit" or username == "Exit": #If the user enters "exit" for their username, close the program
        exit()
    
    for i in range(len(credentialsList)): #Find user's unique salt based on the username they entered
        if username == credentialsList[i]['username']:
            passwordSalt = credentialsList[i]['salt']
                
    hashedPassword = hashlib.sha256((input("Enter your password: ") + individualPasswordSalt + globalPasswordSalt).encode()) #Takes user's password, adds salt, then Hashes password with SHA-256 encryption
    
    for i in range(len(credentialsList)): #Check if hashed password is in "Credentials.txt", if so, then check if inputted username matches for that user, if both are true, log user in
        if hashedPassword.hexdigest() == credentialsList[i]['password'] and username == credentialsList[i]['username']:
            validInfo = True
            currentRole = credentialsList[i]['role']
    
    totalTries += 1
        
    if validInfo: #Once valid credentials are entered, search "Roles.txt" for user's role message.
        for j in range(len(rollsList)):
            if currentRole == rollsList[j]['role']:
                print(rollsList[j]['message'])
                while logout != "logout": #Repeatably ask user to type "logout" to logout of their account
                    logout = input("Type \"logout\" to logout: ")
                totalTries = 0
            if j == len(rollsList) - 1: #If user's role isn't found in "Roles.txt", display error message
                print("System role not found. Contact system administrator.")

    else:    
        print("Invalid Credentials")
        print("Tries remaining: ", allowedTries - totalTries)
            
    if totalTries >= allowedTries: #Exit program if too many failed login attempts are entered in a row (Default = 3)
        input("Too many login attempts. Press \'Enter\' to close program.")
        exit()
```

Credentials.txt
```python
{'username': 'griffinKeyes', 'password': '438c7bae7b312cfcd5f6260236346f9fb04acaa69fed60013ef14dcc82e84595', 'salt': '7W5gBDZTv!$8Nc2%4$Exzyf9k&W56JWe*%24o32FM&!uikMaLUja7WGqJ$fgM*NSpkQbq2DNF@jT5Vq6Tty7uP3iuukh&5d&AxH@km5rXpQB$nXpnbKczQdksMMXz*Jf', 'role': 'zookeeper'}
{'username': 'rosarioDawson', 'password': 'ced5282509ce44a4d42f34979cd713e7e119e1a7b47fedc1c5d3966f45e0d47e', 'salt': 'l0P5&BN4yGEYuBJcQHNy03XBkN$26I#gnzdzwHJia2Akjl9f3x7EZJHP4ZMBU42LmoIKj*QX03oGy#@7f6ZfKzTLKPpYGmpS83XS!t8@5d#4^0CA7ZZtS%R%h9yr*23s', 'role': 'admin'}
{'username': 'bernieGorilla', 'password': '1125a825d878fa0f61185e675f116b08547e4d30db40706876f6764fa819fee6', 'salt': 'z5c$s9*q#m1GAvrUmIZ$Osc&Z6nluOQPQN2Vup*LwJQEIImuOMv!CMvh$zR2!O22PeuFB@Z&QdCLnq5F34AvDoh*Q&^BcUAORl%2nPJ3Q1P#3oQaTTUQ!I!^LNC62PlH', 'role': 'veterinarian'}
{'username': 'donaldMonkey', 'password': 'a06997284d4ddb3a6cb2a867c54c8d8519115630b49233db26f2e4325b9979b2', 'salt': '4GYfkbtx1oHI8jeW6ANbR7WgasA3BMcv$5V9$B&MIPNp20$RSjK!m4k!wwX@%^ytJsS2h2m!vRGtZ!%zM&$vdm@u2G^wLuut4IPgmG9K^0l5X6FGZP^fIEbULgEmZ*2X', 'role': 'zookeeper'}
{'username': 'jeromeGrizzlybear', 'password': 'f96e082a8efe786c6948445401ba8ebe283cc0f3a69167ac1947f9c5cd58d380', 'salt': 'pz5mc21*rZkQTmHd0dyuGO4E*z^$fVGaZloN8h%6EOJjAs%Lr@x#%RuKdH5LL@MH0feV7RrjrMUQfCL4TBM0N3L&!NCv1NYbp^XHju4NE99kAHu&qit^J8SvQpXX#RRG', 'role': 'veterinarian'}
{'username': 'bruceGrizzlybear', 'password': '0749eeaf62f49edeec49700d1bd7742824784f1907ebbe2d81fb6e1aa572e24c', 'salt': '50%TPGU2fp*T!N28oW^T6taI!3a6Za7zzuGNJ3ozna6#nm*kTxS$mSYwLmp%9TQph8pMTv%a24AbvG87WyDpN04Gzu1P%L^yg00!1Ug@&PzQz3Ycm8nIpQpaA$DgUhq@', 'role': 'admin'}
```

Roles.txt
```python
{'role':'admin', 'message': 'Hello, System Admin! As administrator, you have access to the zoo main computer system. This allows you to monitor users in the system and their roles.'}
{'role': 'zookeeper', 'message': 'Hello, Zookeeper! As zookeeper, you have access to all of the animal information and their daily monitoring logs. This allows you to track their feeding habits, habitat conditions, and general welfare.'}
{'role': 'veterinarian', 'message': 'Hello, Veterinarian! As veterinarian, you have access to all of the animal health records. This allows you to view each animal medical history and current treatments/illnesses (if any), and to maintain a vaccination log.'}
```

**Narrative**

I chose to include this item in my ePortfolio because the original version of the program used an ineffective method to store and retrieve data. In the original program, all user information such as usernames, passwords, and roles, were stored in individual arrays. When a user would attempt to log in, first the password they entered would be hashed and compared to all the passwords in the password array. If a match were found, the position of that password in the array would be saved and then the username at that position in the username array would be compared to the username the user entered. Finally, if the usernames matched, then the role in the role array at that position would be checked to output a message corresponding to the correct role. While this did work on a technical level, it is far from the best approach to handle the needs of this program, as a single mistake while modifying the arrays would cause a desync between entries, potentially causing login credentials to swap between users. To improve the program, I changed it so that individual user’s information was stored within a dictionary, meaning each dictionary contains one username, one password, and one role. This means user credentials can be modified by changing the already existing dictionary for them, new users can be added by creating a new dictionary for them, and old users can be deleted by deleting their dictionary. As each dictionary corresponds to a single user, there is no risk of an error while modifying one user’s dictionary affecting the other user’s dictionaries, helping to make the program more secure. Additionally, I also decided to improve the level of security used to store user’s passwords by changing the hashing function from the now unsecure MD5 hash to the currently secure SHA-256 hash. The SHA-256 hash has a potential 2^256 outputs, which means when given an SHA-256 hashed password, attempting to find another password that produces the same hash is currently impossible. Additionally, I included two 128-character salts to all the user’s passwords, one unique to each user and one added globally, meaning even if a user used an extremely simple password like “password”, the attacker would still have to guess up to 94^257 random passwords before brute forcing it, a feat that is currently impossible to accomplish. In the end, by making this enhancement, I have showcased my understanding of data structures and how some data structures are better suited for specific tasks than others, and my understanding of securely storing sensitive information and making data resistant to attacks.

When I first chose this enhancement, I didn’t actually know what data structures Python included, or which one would be the best choice for storing user credentials, I simply knew my old series of arrays was not the best choice. Before I began the enhancement, I had to perform research on the different Python data structures and determine which one I wanted to use. Since I hadn’t used dictionaries before, I also had to research how to work with the data structure so I could actually use the data being stored in it. While this process ended up not being particularly difficult, I think it highlights the fact that nobody understands every aspect of a programming language, so being able to look up information and implement it in your programs is an important part of being a programmer. The biggest challenge I faced was figuring out how to correctly read data from a dictionary, but thanks to the vast resources available on the internet, this challenge wasn’t too big of a hinderance as I found many different examples showing different ways to utilize them, including the way I needed to for my program.

**Artifacts**

[Home Page](./Home.md)

[Software Design and Engineering](./ArtifactOne.md)

[Databases](./ArtifactThree.md)
