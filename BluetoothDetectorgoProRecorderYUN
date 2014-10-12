#include <SoftwareSerial.h>
#include <Bridge.h>
#include <HttpClient.h>
#include <FileIO.h>

//This is the main timer for the bluetooth search cycle
unsigned long lastTimezz;
unsigned long intervalzz = 5000;
//The timer for the bluetooth as it sends in data, currently waits 9 seconds
unsigned long lastTime;
unsigned long interval = 9000;
//Number of serial numbers expected to be picked up
String devices[60];
//Device Counter
int amountDev = 0;
//Boolean to stop device being stated as new if it is not.
bool notNew = false;
//Digital Pin 8 goes to bluetooth TX and Digital pin 5 goes to bluetooth RX
SoftwareSerial bluetooth(8, 5);
//char dateTime[20];

void setup()
{
  //  while (!Serial) ;
  Serial.begin(9600);  // Begin the serial monitor at 9600bps
  bluetooth.begin(115200);
  bluetooth.print("$");  // Print three times individually
  bluetooth.print("$");
  bluetooth.print("$");  // Enter command mode
  delay(100);  // Short delay, wait for the Mate to send back CMD
  bluetooth.println("U,9600,N");  // Temporarily Change the baudrate to 9600, no parity
  bluetooth.begin(9600);  // Start bluetooth serial at 9600
  delay(200);
  Serial.println("Started....");
  bluetooth.print("$$$");//Enters command mode at 9600

  // Bridge takes about 30-60 seconds to start up
  pinMode(13, OUTPUT);
  digitalWrite(13, LOW);
  Bridge.begin();
  digitalWrite(13, HIGH);
  Serial.println("Serial port connected.");
}

void loop()
{
  unsigned long currentTimezz = millis();
  if ((currentTimezz - lastTimezz) > intervalzz) {

    Serial.println("LETS START THE PROGRAM!");
    bluetooth.print("$$$");//Enters into command mode
    delay(200);
    bluetooth.println("IN,3");//Inquires for 3 seconds
    char *id = readSerial();
    String dataString; //Here is to save to the file
    if (strlen(id) >= 20)
    {
      char *str; //Breaks the bluetooth data into each line
      while ((str = strtok_r(id, "\n", &id)) != NULL)
      {
        if (strlen(str) > 13) //makes sure the line is over 13 characters long
        {
          Serial.print("This String is: ");
          Serial.println(str);
          dataString += String(str) + "\n";//Adds string to the txt file
          for (int i = 0; i <= amountDev; i++)
          {
            char charBuf[15];
            devices[i].toCharArray(charBuf, 15);
            //Checks through the saved strings to see if the serial has been recorded
            if (strncmp(str, charBuf, 15) == 0) {
              Serial.print("Already saved device with id: ");
              Serial.println(str);
              notNew = true;// Makes sure to not save it as a new one
            }
          }
          if (!notNew)
          {
            Serial.print("Found new device! With id: ");
            Serial.println(str);
            dataString += "This is new! \n";
            devices[amountDev] = String(str);
            amountDev ++;

            //Tell GOPRO to turn on and reord video for around 12 seconds
            HttpClient client;
            client.get("http://10.5.5.9/bacpac/PW?t=<GOPROPASSWORD>&p=%01");
            Serial.println("Turn on Camera.");
            delay(6500);              // wait for a second
            client.get("http://10.5.5.9/bacpac/SH?t=<GOPROPASSWORD>&p=%01");
            Serial.println("Record video.");
            delay(15000);              // wait for a second
            client.get("http://10.5.5.9/bacpac/SH?t=<GOPROPASSWORD>&p=%00");
            Serial.println("Stop video.");
            delay(3000);              // wait for a second
            client.get("http://10.5.5.9/bacpac/PW?t=<GOPROPASSWORD>&p=%00");
            Serial.println("Turn off.");
            delay(3000);
          }
          notNew = false;// Make it false again
        }
      }
      File dataFile = FileSystem.open("/mnt/sd/arduino/testright.txt", FILE_APPEND);

      // if the file is available, write to it:
      if (dataFile) {
        dataString += getTimeStamp() + "\n";
        dataFile.println(dataString + "End of check \n \n");
        dataFile.close();
        // print to the serial port too:
        Serial.println(dataString);
      }
      // if the file isn't open, pop up an error:
      else {
        Serial.println("error opening datalog.txt");
      }
    }
    else if (strlen(id) <= 10)
    { // we know there is no bluetooth device detected
      Serial.println("No device found");
    }
    bluetooth.println("---");//Get out of command mode
    unsigned long currentTimezz = millis();//Reset timer
    lastTimezz = currentTimezz;
  }
  clearAll();//Removes any leftover sent data from the bluetooth
}


void clearAll() {
  for (int i = 0; i < bluetooth.available(); i++) {
    bluetooth.read();
  }
}

char* readSerial() {
  unsigned long currentTime = millis();
  // Check if its already time for our process
  lastTime = currentTime;

  char buffer[150];//Only 150 characters allowed to be picked up/sent by the bluetooth
  char input;
  int j = 0;
  while (bluetooth.available() || (currentTime - lastTime) < interval) {
    currentTime = millis();
    if (bluetooth.available())
    {
      input = (char)bluetooth.read();
      lastTime = currentTime;
      buffer[j] = input;
      Serial.print(buffer[j]);
      buffer[j + 1] = '\0';
      j++;
    }
  }
  return buffer;
}

// This function returns a string with the time stamp
String getTimeStamp() {
  String result;
  Process time;
  // date is a command line utility to get the date and the time
  // in different formats depending on the additional parameter
  time.begin("date");
  time.addParameter("+%D-%T");  // parameters: D for the complete date mm/dd/yy
  //             T for the time hh:mm:ss
  time.run();  // run the command

  // read the output of the command
  while (time.available() > 0) {
    char c = time.read();
    if (c != '\n')
      result += c;
  }
  return result;
}
