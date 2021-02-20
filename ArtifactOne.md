**Artifact One**

(Demonstrates Software Design and Engineering, Algorithms, and Data Structures)

For this artifact, I took the Authentication System that I created in Java for my IT 145: Foundation in Application Development course and ported it to Python while updating
the code to be more efficient and easier to understand. As one of my first Java programs, the original contained many inefficiencies and poor design choices, including
splitting the code into two seperate files, storing data within the program itself, and storing user data in multiple arrays where desyncs could easily occure.

AuthenticationSystem.java
```Java
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
```Java
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
