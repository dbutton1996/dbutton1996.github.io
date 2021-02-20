**Artifact One**

**(Demonstrates Software Design and Engineering)**

For this artifact, I took the Authentication System that I created in Java for my IT 145: Foundation in Application Development course and ported it to Python while updating
the code to be more efficient and easier to understand. The purpose of this program is to prompt the user for a username and password, and then display a specific line of text depending on the credentials entered. Additionally, if invalid credentials are entered three times in a row, then the program will exit for security purposes. As one of my first Java programs, the original contained many inefficiencies, such as the unnecessary variables "looping" and "password", and poor design choices such as splitting the program into two seperate Java files and hard coding user data in the Java program itself.

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

When porting this program to Python, I made care to reduce the number of variables to those required for the program to run. Additionally, all the code was condensed into a single file and the user data was taken out of the program and instead is stored in two seperate .txt files.

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
```
{'username': 'griffinKeyes', 'password': '438c7bae7b312cfcd5f6260236346f9fb04acaa69fed60013ef14dcc82e84595', 'salt': '7W5gBDZTv!$8Nc2%4$Exzyf9k&W56JWe*%24o32FM&!uikMaLUja7WGqJ$fgM*NSpkQbq2DNF@jT5Vq6Tty7uP3iuukh&5d&AxH@km5rXpQB$nXpnbKczQdksMMXz*Jf', 'role': 'zookeeper'}
{'username': 'rosarioDawson', 'password': 'ced5282509ce44a4d42f34979cd713e7e119e1a7b47fedc1c5d3966f45e0d47e', 'salt': 'l0P5&BN4yGEYuBJcQHNy03XBkN$26I#gnzdzwHJia2Akjl9f3x7EZJHP4ZMBU42LmoIKj*QX03oGy#@7f6ZfKzTLKPpYGmpS83XS!t8@5d#4^0CA7ZZtS%R%h9yr*23s', 'role': 'admin'}
{'username': 'bernieGorilla', 'password': '1125a825d878fa0f61185e675f116b08547e4d30db40706876f6764fa819fee6', 'salt': 'z5c$s9*q#m1GAvrUmIZ$Osc&Z6nluOQPQN2Vup*LwJQEIImuOMv!CMvh$zR2!O22PeuFB@Z&QdCLnq5F34AvDoh*Q&^BcUAORl%2nPJ3Q1P#3oQaTTUQ!I!^LNC62PlH', 'role': 'veterinarian'}
{'username': 'donaldMonkey', 'password': 'a06997284d4ddb3a6cb2a867c54c8d8519115630b49233db26f2e4325b9979b2', 'salt': '4GYfkbtx1oHI8jeW6ANbR7WgasA3BMcv$5V9$B&MIPNp20$RSjK!m4k!wwX@%^ytJsS2h2m!vRGtZ!%zM&$vdm@u2G^wLuut4IPgmG9K^0l5X6FGZP^fIEbULgEmZ*2X', 'role': 'zookeeper'}
{'username': 'jeromeGrizzlybear', 'password': 'f96e082a8efe786c6948445401ba8ebe283cc0f3a69167ac1947f9c5cd58d380', 'salt': 'pz5mc21*rZkQTmHd0dyuGO4E*z^$fVGaZloN8h%6EOJjAs%Lr@x#%RuKdH5LL@MH0feV7RrjrMUQfCL4TBM0N3L&!NCv1NYbp^XHju4NE99kAHu&qit^J8SvQpXX#RRG', 'role': 'veterinarian'}
{'username': 'bruceGrizzlybear', 'password': '0749eeaf62f49edeec49700d1bd7742824784f1907ebbe2d81fb6e1aa572e24c', 'salt': '50%TPGU2fp*T!N28oW^T6taI!3a6Za7zzuGNJ3ozna6#nm*kTxS$mSYwLmp%9TQph8pMTv%a24AbvG87WyDpN04Gzu1P%L^yg00!1Ug@&PzQz3Ycm8nIpQpaA$DgUhq@', 'role': 'admin'}
```

Roles.txt
```
{'role':'admin', 'message': 'Hello, System Admin! As administrator, you have access to the zoo main computer system. This allows you to monitor users in the system and their roles.'}
{'role': 'zookeeper', 'message': 'Hello, Zookeeper! As zookeeper, you have access to all of the animal information and their daily monitoring logs. This allows you to track their feeding habits, habitat conditions, and general welfare.'}
{'role': 'veterinarian', 'message': 'Hello, Veterinarian! As veterinarian, you have access to all of the animal health records. This allows you to view each animal medical history and current treatments/illnesses (if any), and to maintain a vaccination log.'}
```

**Narrative**

I included this artifact to show my ability to create programs in both Java and Python as well as to show how my programming abilities have improved. Originally, the program was created in Java, and while the program worked as intended, there were some inefficiencies and poor design choices such as the user credentials being stored within the Java program, and important functions being split between two Java files when one would have sufficed. By translating this program from Java to Python, I have shown that I am capable of programming in both languages, a useful skill in the programming world as both languages are widely used by companies. Additionally, by reworking the program to be more efficient and intuitive, I have shown that my overall programming abilities have improved over the years. While initially the program consisted of two Java files, one of which contained code, user credentials, and the messages displayed after logging in, my improved program consists of a single Python file, as well as two data files. The Python file contains all the code required to run the program, and the two data files contain the user credentials and the messages that will be displayed. This means that if the programmers needed to add, remove, or update users or messages in the system, they can do so by editing the data files without having to edit the Python file. Additionally, in the program file, I have removed some variables that didn’t need to be included for the code to function correctly, such as a base password variable that stored the password the user entered so it could be hashed, at which point it would be saved as a different variable. In the improved version, when the user inputs their password, it is immediately hashed before being saved to a variable, making the program slightly more efficient. Since this was such a small program to begin with there weren’t any major efficiencies I could create, but small improvements such as removing a variable for the base passwords can build up over time in larger programs and make them run noticeably faster.

I learned a few things while working on this enhancement, the biggest of which is the difficulty of translating a program from one programming language to another. While most programming languages fundamentally work similarly, using variables, if statements, loops, and functions to perform the action the programmer wants, each programming language has its own unique quirks that can make changing the language for a program more difficult than one might initially expect. In the case of going from Java to Python, the most noticeable difference is how each language approaches syntax. In Java, the end of every line, if statement, loop, and function must be denoted with closing brackets or semicolons, and if one is missed the program will crash. In Python however, the syntax is handled almost entirely though the indentation of each line. Individual lines can just end without any special symbol announcing their end. If statements, loops, and functions must be started with a colon, but every line after simply must be indented once to show it is part of the that statement. Once the code is no longer supposed to be in the statement, you simply stop indenting it. Because of this fundamental difference between Java and Python, I found the best solution to translate code from one to the other was to rewrite the code from scratch using the original pseudocode, rather than copying the code from Java to Python and modifying each line, statement, loop, and function to fit the syntax of Python. Aside from changing the syntax to work in Python, the biggest challenge I faced was using separate files to store the data rather than incorporating the data in the program file itself. To make this function work correctly, I needed to research how Python handles importing and exporting data, and how that data should be stored to best be imported and used. In the end, my solution was to store data in .txt files in the form of Python dictionaries. Then when the data is imported into the Python program to be used, it is placed in order inside of a list. This allowed me to iterate through each dictionary in the list to find the correct user information and to output the appropriate message depending on the user’s role. When I initially tried this approach, I ran into a problem. Every time I would try to use the data once it was imported into my program, my program would crash. After some debugging, I realized that had made a mistake when placing my data within the .txt files, and each line contained a dictionary within a list, so when the data was imported into the program it was done so as a dictionary in a list in a second list. After realizing and fixing this mistake, I was then able to import data from a .txt file without my program crashing.

[Home Page](./Home.md)

[Algorithms and Data Structures](./ArtifactTwo.md)

[Databases](./ArtifactThree.md)
