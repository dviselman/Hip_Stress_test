Hip_Stress_test
===============

Hit stress test




//motor x control -> 9
//motor y control -> 8
//
//--Darlington Transistor Array(http://www.ti.com/lit/ds/symlink/uln2803a.pdf)--
//all B pins -> GND
//C pins(1-6) -> motor GND(brown)d
//COM -> transister(collecter)
//
//--bc517 transister--
//collector -> COM
//base -> 100 ohm resister -> pin 7
//emitter -> GNDs
//
//hall effect sensor(short side facing you)
//1 -> 5V
//2 -> GND
//3 -> A0 thru A5

//SD shield
//miso -> 50
//sck -> 52
//mosi -> 51
//cs -> 53

//remember to connect all the grounds
//this code works on the arduino mega2560 only
//if SD card is missing or misconfigured, the 'L' LED will light up
//bug when a leg times out, the 'L' LED will blink
//each leg you add will need another transistor

#include <Servo.h>
#include <SD.h>
#include <SPI.h>





Servo servo_x;
Servo servo_y;

Servo servo_x1;
Servo servo_y1;

Servo servo_x2;
Servo servo_y2;

Servo servo_x3;
Servo servo_y3;


static const int PIN_X = 9; //first leg motor x
static const int PIN_Y = 8;//first leg motor y

static const int PIN_X_1 = 11;//second leg motor x
static const int PIN_Y_1 = 10;//second leg motor y

static const int PIN_X_2 = 13;//third leg motor x
static const int PIN_Y_2 = 12;//third leg motor y

static const int PIN_X_3 = 7;//third leg motor x
static const int PIN_Y_3 = 6;//third leg motor y


static int ENABLE[6] = {
  31,33,35,37,39,41}; // pin 31 thru 41 odd pins only are leg enabler pins

boolean triggered[6] = 
{
  true,true,true,true,true,true};// array  index corresponding to each hall effect sensor.


static const int CENTER = 90;
static const int UPPER = 130;
static const int LOWER = 50;

int halleffectsensor0=22;
int halleffectsensor1=24;
int halleffectsensor2=26;
int halleffectsensor3=28;

const int chipSelect = 53;//cs

File datafile;

unsigned int readsensor(int *sensor)
{ 
  if(triggered[0]==true)//
    sensor[0] = digitalRead(halleffectsensor0);//hall effect sensor 1
  else sensor[0]=0;//

  if(triggered[1]==true)
    sensor[1]= digitalRead(halleffectsensor1);//hall effect sensor 2
  else sensor[1]=0;

  if(triggered[2]==true)
    sensor[2]= digitalRead(halleffectsensor2);//hall effect sensor 2
  else sensor[2]=0;

  if(triggered[3]==true)
    sensor[3]= digitalRead(halleffectsensor3);//hall effect sensor 2
  else sensor[3]=0;

  return     sensor[0] + sensor[1] + sensor[2] + sensor[3];// + sensor[4] + sensor[5];
}

void setup()
{
  Serial.begin(57600);
  while(!Serial)
    delay(1000);
  Serial.println("Serial Test...");

  servo_x.attach(PIN_X);
  servo_y.attach(PIN_Y); 

  servo_x1.attach(PIN_X_1);
  servo_y1.attach(PIN_Y_1);

  servo_x2.attach(PIN_X_2);
  servo_y2.attach(PIN_Y_2); 

  servo_x3.attach(PIN_X_3);
  servo_y3.attach(PIN_Y_3); 

  pinMode(halleffectsensor0,INPUT_PULLUP);// hall effect sensor 0
  pinMode(halleffectsensor1,INPUT_PULLUP);//hall effect sensor 1
  pinMode(halleffectsensor2,INPUT_PULLUP);//hall effect sensor 2
  pinMode(halleffectsensor3,INPUT_PULLUP);//hall effect sensor 3

  for(int i = 0;i < 6;i++)
  {
    pinMode(ENABLE[i],OUTPUT); //enable
    digitalWrite(ENABLE[i],HIGH);
  }

  pinMode(53, OUTPUT); //needed for SD
  pinMode(13,OUTPUT); //LED

  if (!SD.begin(chipSelect)) 
  {         
    Serial.println("Card Failed");
    digitalWrite(13,HIGH);
    while(1);
  }

  if(!(datafile = SD.open("DATALOG.txt", FILE_WRITE)))
  {
    digitalWrite(13,HIGH);
    Serial.println("Card Failed");
    while(1);
  }
  datafile.println("Logging started...");
  Serial.println("Logging Started..."); 
}

void loop()
{
  int timeout = 0;
  int sensor[6];//6 legs

  servo_x.write(UPPER);
  servo_y.write(UPPER);

  servo_x1.write(UPPER);
  servo_y1.write(UPPER);

  servo_x2.write(UPPER);
  servo_y2.write(UPPER);

  servo_x3.write(UPPER);
  servo_y3.write(UPPER);

  delay(1000);


  while(readsensor(sensor) && timeout < 15)
  {
    digitalWrite(13,HIGH);



    if(sensor[0] && ENABLE[0])  //hall effect sensor didn't trigger and leg hasn't timed out
    {
      if(timeout==15)//timeout test-----------------
      {
        // triggered[0]=false;
        servo_x.detach();
        servo_y.detach();
        datafile.println(String(millis()/1000)+": A0 did no reach target in time.");
      }
    }



    if(sensor[1] && ENABLE[1])
    { 
      if(timeout==15)//timeout test-----------------
      {  

        datafile.println(String(millis()/1000)+": A1 did no reach target in time.");
        // triggered[1]=false;
        servo_x1.detach();
        servo_y1.detach();  
      }
    }


    if(sensor[2] && ENABLE[2])
    {
      if(timeout==15)//timeout test-----------------
      {  
        datafile.println(String(millis()/1000)+": A2 did no reach target in time.");
        // triggered[2]=false;
        servo_x2.detach();
        servo_y2.detach();  
      }
    }
    
    
    if(sensor[3] && ENABLE[3])
    {
      if(timeout==15)//timeout test-----------------
      {  
        datafile.println(String(millis()/1000)+": A3 did no reach target in time.");
        servo_x3.detach();
        servo_y3.detach();  
      }
    }




    /* if(sensor[4] && ENABLE[4])
     datafile.println(String(millis()/1000)+": A4 did no reach target in time.");
     if(sensor[5] && ENABLE[5])
     datafile.println(String(millis()/1000)+": A5 did no reach target in time.");
     */

    datafile.flush();

    timeout++;
    delay(100);  
    digitalWrite(13,LOW);
  }
  
  if(timeout == 15)
  {
    int i;
    for(i = 0;sensor[i] == 0;i++); //which sensor didn't trigger
    if(ENABLE[i] != 0)//hasn't timed out
    {
      digitalWrite(ENABLE[i],LOW);
      datafile.println("A"+String(i)+": timeout!");
      ENABLE[i] = 0;
      triggered[i]=false;
    }
  }

  timeout = 0;

  servo_x.write(UPPER);         
  servo_y.write(LOWER);

  servo_x1.write(UPPER);         
  servo_y1.write(LOWER);

  servo_x2.write(UPPER);         
  servo_y2.write(LOWER);

  servo_x3.write(UPPER);         
  servo_y3.write(LOWER);

  delay(1000);

  /*
  while(readsensor(sensor) && timeout < 15)
   {
   digitalWrite(13,HIGH);
   
   
   
   if(sensor[0] && ENABLE[0])
   {
   
   if(timeout==15)//timeout test-----------------
   {
   //   triggered[0]=false;
   servo_x.detach();
   servo_y.detach();
   
   datafile.println(String(millis()/1000)+": A0 did no reach target in time.");
   }
   }
   
   
   
   if(sensor[1] && ENABLE[1])
   {
   if(timeout==15)//timeout test-----------------
   {  
   
   datafile.println(String(millis()/1000)+": A1 did no reach target in time.");
   // triggered[1]=false;
   servo_x1.detach();
   servo_y1.detach();  
   }
   }
   
   
   
   
   if(sensor[2] && ENABLE[2])
   {
   if(timeout==15)//timeout test-----------------
   {  
   datafile.println(String(millis()/1000)+": A2 did no reach target in time.");
   // triggered[2]=false;
   servo_x2.detach();
   servo_y2.detach();  
   }
   }
  /* if(sensor[3] && ENABLE[3])
   datafile.println(String(millis()/1000)+": A3 did no reach target in time.");
   if(sensor[4] && ENABLE[4])
   datafile.println(String(millis()/1000)+": A4 did no reach target in time.");
   if(sensor[5] && ENABLE[5])
   datafile.println(String(millis()/1000)+": A5 did no reach target in time.");
   datafile.flush();
   timeout++;
   delay(100);  
   digitalWrite(13,LOW);
   }
   if(timeout == 15)
   {
   int i;
   for(i = 0;sensor[i] == 0;i++);
   if(ENABLE[i] != 0)
   {
   digitalWrite(ENABLE[i],LOW);
   datafile.println("A"+String(i)+": timeout!");
   ENABLE[i] = 0;
   triggered[i]=false;
  
   }
   }  
   */
  // timeout = 0;

  servo_x.write(LOWER);
  servo_y.write(LOWER);

  servo_x1.write(LOWER);
  servo_y1.write(LOWER);

  servo_x2.write(LOWER);
  servo_y2.write(LOWER);

  servo_x3.write(LOWER);
  servo_y3.write(LOWER);

  delay(1000);
  /*
  while(readsensor(sensor) && timeout < 15)
   {
   digitalWrite(13,HIGH);
   if(sensor[0] && ENABLE[0])
   {
   if(timeout==15)//timeout test-----------------
   {
   //  triggered[0]=false;
   servo_x.detach();
   servo_y.detach();
   
   datafile.println(String(millis()/1000)+": A0 did no reach target in time.");
   }
   }
   if(sensor[1] && ENABLE[1])
   {
   if(timeout==15)//timeout test-----------------
   {  
   
   datafile.println(String(millis()/1000)+": A1 did no reach target in time.");
   // triggered[1]=false;
   servo_x1.detach();
   servo_y1.detach();  
   }
   }
   
   if(sensor[2] && ENABLE[2]){
   if(timeout==15)//timeout test-----------------
   {  
   datafile.println(String(millis()/1000)+": A2 did no reach target in time.");
   // triggered[2]=false;
   servo_x2.detach();
   servo_y2.detach();  
   }
   }
   
   
  /* if(sensor[3] && ENABLE[3])
   datafile.println(String(millis()/1000)+": A3 did no reach target in time.");
   if(sensor[4] && ENABLE[4])
   datafile.println(String(millis()/1000)+": A4 did no reach target in time.");
   if(sensor[5] && ENABLE[5])
   datafile.println(String(millis()/1000)+": A5 did no reach target in time.");
   
   
   datafile.flush();
   timeout++;
   delay(100);  
   digitalWrite(13,LOW);
   }
   if(timeout == 15)
   {
   int i;
   for(i = 0;sensor[i] == 0;i++);
   if(ENABLE[i] != 0)
   {
   digitalWrite(ENABLE[i],LOW);
   datafile.println("A"+String(i)+": timeout!");
   ENABLE[i] = 0;
   triggered[i]=false;
   
   
   
   }
   }
   */
  //timeout = 0;

  servo_x.write(LOWER);
  servo_y.write(UPPER);

  servo_x1.write(LOWER);
  servo_y1.write(UPPER);

  servo_x2.write(LOWER);
  servo_y2.write(UPPER);

  servo_x3.write(LOWER);
  servo_y3.write(UPPER);

  delay(1000);
  /*
  while(readsensor(sensor) && timeout < 15)
   {
   digitalWrite(13,HIGH);
   if(sensor[0] && ENABLE[0])
   {
   if(timeout==15)//timeout test-----------------
   {
   //   triggered[0]=false;
   servo_x.detach();
   servo_y.detach();
   datafile.println(String(millis()/1000)+": A0 did no reach target in time.");
   }
   }
   if(sensor[1] && ENABLE[1]){
   if(timeout==15)//timeout test-----------------
   {  
   datafile.println(String(millis()/1000)+": A1 did no reach target in time.");
   //  triggered[1]=false;
   servo_x1.detach();
   servo_y1.detach();  
   }
   }
   
   if(sensor[2] && ENABLE[2]){
   if(timeout==15)//timeout test-----------------
   {  
   datafile.println(String(millis()/1000)+": A2 did no reach target in time.");
   // triggered[2]=false;
   servo_x2.detach();
   servo_y2.detach();  


}
   
   }
  /* if(sensor[3] && ENABLE[3])
   datafile.println(String(millis()/1000)+": A3 did no reach target in time.");
   if(sensor[4] && ENABLE[4])
   datafile.println(String(millis()/1000)+": A4 did no reach target in time.");
   if(sensor[5] && ENABLE[5])
   datafile.println(String(millis()/1000)+": A5 did no reach target in time.");
   
   datafile.flush();
   timeout++;
   delay(100);  
   digitalWrite(13,LOW);
   }
   if(timeout == 15)
   {
   int i;
   for(i = 0;sensor[i] == 0;i++);
   if(ENABLE[i] != 0)
   {
   
   digitalWrite(ENABLE[i],LOW);
   datafile.println("A"+String(i)+": timeout!");
   ENABLE[i] = 0;
   triggered[i]=false;
   

   
   
   }
   } 
   */
  // timeout = 0;

}



