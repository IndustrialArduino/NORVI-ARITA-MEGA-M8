/*
 * Iconic Norvi Standard K8
 * 
 * This program can be used to interface all features in the Norvi Standard K8 controller
 * 
 * 1. Interface to all digital inputs
 * 2. Interface to all Analog inputs
 * 3. Control Relay outputs
 * 4. Control Transistor PWM outputs
 * 5. Setup Ethernet connection and set the Norvi as a webserver
 */

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <SPI.h>
#include <SD.h>
#include <Ethernet2.h>
#include <avr/pgmspace.h>

// All digital Inputs
#define IN1 19
#define IN2 18
#define IN3 26
#define IN4 27
#define IN5 28
#define IN6 29
#define IN7 30
#define IN8 31
#define IN9 32
#define IN10 33
#define IN11 34
#define IN12 35
#define IN13 36
#define IN14 37

// All ouputs
#define O1 4
#define O2 5
#define O3 2
#define O4 3
#define O5 6
#define O6 7
#define O7 8  
#define O8 9  
#define O9 53
#define O10 10
#define O11 11
#define O12 12

// Front face buttons
#define b4 22 // down
#define b3 23 // right
#define b2 24 // left
#define b1 25 // up

// defining the OLED
#define OLED_RESET 40
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// SD Paramerters
#define SD_chipSelect 46
Sd2Card card;
SdVolume volume;
SdFile root;



// All needed global variables
bool but1 = 0, but2 = 0, but3 = 0, but4 = 0, blinknow = 0, valsChanged=0, changeWeb = 0;
bool b1pressed = 0, b2pressed = 0, b3pressed = 0, b4pressed = 0, configured = 0;

String currentIP="";

bool in[14] = {};
bool out[12] = {};
uint8_t pwm1 = 0, pwm2 = 0;

int8_t selec = 0, page = 0;
int16_t an0=0, an1=0, an2=0, an3=0, prevIn = 0, prevOut = 0, prevPWM1=0, prevPWM2=0;
int16_t prevan0=0,prevan1=0,prevan2=0,prevan3=0;

int val = 0, pg = 0;

#define REQ_BUF_SZ 100
unsigned int net_stat = 0;
char HTTP_req[REQ_BUF_SZ] = {0}; // buffered HTTP request stored as null terminated string
int req_index = 0;  

// IP, Subnet, and Domain
byte t1 = 192;  byte t2 = 168; byte t3 = 1; byte t4 = 150;
byte t1s = 255; byte t2s = 255; byte t3s = 255; byte t4s = 0; 
byte t1g = 192; byte t2g = 168; byte t3g = 1; byte t4g = 1;

// Ethernet Parameters
#if defined(WIZ550io_WITH_MACADDRESS) // Use assigned MAC address of WIZ550io
;
#else
byte mac[] = {0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xAE};
#endif 
EthernetServer server(80);

// SAVING THE HTML PAGE to FLASH MEMORY to save up dynamic memory
// Html pages sizes up to 50kB can be stored using this method
const char HtmlPage[] PROGMEM  ={ "<!DOCTYPE html>\n <html>\n <head>\n <title>Norvi Ethernet Test Program</title>\n <script>\n var reqval = 1;\n var typ='0';\n function GetSwitchState(){\n var mul='';\n if(reqval>999) {mul='000';}\n else if(reqval>99) {mul='0000';}\n else if(reqval>9) {mul='00000';}\n else if(reqval>=0) {mul='000000';}\n nocache = '&nocache=' + typ + mul + reqval;\n var request = new XMLHttpRequest();\n request.onreadystatechange = function()\n {\n if(this.readyState == 4)\n {\n if(this.status == 200)\n {\n if(this.responseText != null)\n {\n eval(this.responseText);\n }\n }\n }\n }\n request.open('GET', 'ajax_switch' + nocache, true);\n request.send('hello chamith');\n setTimeout('GetSwitchState()', 2000);\n reqval = 1; typ='0';\n }\n \n </script>\n \n <style type='text/css'>\n DIV,UL,OL /* Left */\n {\n margin-top: 0px;\n margin-bottom: 0px;\n }\n h1{\n font-size: 13px;\n font-family: 'Open Sans', sans-serif;\n }\n h2{\n font-size: 17px;\n font-family: 'Open Sans', sans-serif;\n color: #fff;\n width:85%;\n background-color: #808080;\n border: 1px solid #000;\n }\n h3{\n position:relative;\n top:3px;\n height:20px;\n width:60%;\n font-size: 17px;\n font-family: 'Open Sans', sans-serif;\n color: #fff;\n background-color: #808080;\n border: 2px solid #000;\n }\n h4{\n font-size: 25px;\n font-family: 'Open Sans', sans-serif;\n align:center;\n }\n \n .switch {\n position: relative;\n display: inline-block;\n left:5px;\n width: 36px;\n height: 34px;\n }\n .switch input {display:none;}\n \n .slider {\n position: absolute;\n cursor: pointer;\n top: 0;\n left: 0;\n right: 0;\n bottom: 0;\n background-color: #fff;\n content: 'CHECK';\n -webkit-transition: .4s;\n transition: .4s;\n }\n .switch:hover input ~ .slider {\n background-color: #777;\n }\n .switch input:checked ~ .slider {\n background-color: #2196F3;\n }\n .slider:after {\n position:absolute;\n top:25%;\n left:20%;\n font-size:15px;\n content:'ON';\n display:none;\n }\n .switch input:checked ~ .slider:after {\n display: block;\n }\n div#container\n {\n position:relative;\n width: 1114px;\n margin-top: 0px;\n margin-left: auto;\n margin-right: auto;\n text-align:left; \n }\n body {text-align:center;margin:0}\n </style>\n <div id='container'>\n <div style='position:relative; top:0%; left:34%;'><h4>Norvi Ethernet Test Page</h4></div>\n <div id='table1' style='position:absolute; overflow:hidden; left:0px; top:48px; width:1114px; height:126px; z-index:1'>\n <div><div><TABLE bgcolor='#CFCFCF' border=3 cellpadding=0 bordercolorlight='#000000' bordercolordark='#000000' cellspacing=0>\n <TR valign=top> <TD width=92 height=24 valign=middle style='{border-width : 3px; }'><div>   <div align=center><h1>RS485</h1></div>  </div>  </TD>\n <TD colspan=5 width=310 height=24 valign=middle><div>   <div align=center><h1>Analog Inputs (0-20mA / 0-10v)</h1></div> </div>  </TD>\n <TD colspan=15 width=570 height=24 valign=middle><div>    <div align=center><h1>24V Digital Inputs</h1></div> </div>  </TD> \n </TR>\n <TR valign=top>  <TD width=92 height=69 valign=bottom><div>    <div align=center><h1><BR></h1></div> </div>  </TD>\n <TD width=55 height=69 valign=middle><div>  <div align=center><h1>ACOM</h1></div> </div>  </TD> <TD width=65 height=69 valign=bottom><div>    <div align=center><h2 id='a0'>31255</h2></div>    <div align=center><h1>A0</h1></div> </div>  </TD>\n <TD width=65 height=69 valign=bottom><div>  <div align=center><h2 id='a1'>31255</h2></div>  <div align=center><h1>A1</h1></div> </div>  </TD>\n <TD width=65 height=69 valign=bottom><div>  <div align=center><h2 id='a2'>156</h2></div>  <div align=center><h1>A2</h1></div> </div>  </TD>\n <TD width=65 height=69 valign=bottom><div>  <div align=center><h2 id='a3'>31255</h2></div>  <div align=center><h1>A3</h1></div> </div>  </TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in0'></h3></div>  <div align=center><h1>IN0</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in1'></h3></div>  <div align=center><h1>IN1</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in2'></h3></div>  <div align=center><h1>IN2</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in3'></h3></div>  <div align=center><h1>IN3</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in4'></h3></div>  <div align=center><h1>IN4</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in5'></h3></div>  <div align=center><h1>IN5</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in6'></h3></div>  <div align=center><h1>IN6</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in7'></h3></div>  <div align=center><h1>IN7</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in8'></h3></div>  <div align=center><h1>IN8</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in9'></h3></div>  <div align=center><h1>IN9</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div>  <div align=center><h3 id='in10'></h3></div> <div align=center><h1>IN10</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div> <div align=center><h3 id='in11'></h3></div> <div align=center><h1>IN11</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div> <div align=center><h3 id='in12'></h3></div> <div align=center><h1>IN12</h1></div></div></TD>\n <TD width=37 height=69 valign=bottom><div> <div align=center><h3 id='in13'></h3></div> <div align=center><h1>IN13</h1></div></div></TD>\n <TD width=49 height=69 valign=middle><div> <div align=center><h1>DCOM</h1></div> </div>  </TD>\n </TR>\n </TABLE>\n </div>\n </div></div>\n \n \n <div id='table2' style='position:absolute; overflow:hidden; left:0px; top:363px; width:1114px; height:128px; z-index:0'>\n <div>\n <div>\n <TABLE  bgcolor='#CFCFCF' border=3 cellpadding=0 bordercolorlight='#000000' bordercolordark='#000000' cellspacing=0>\n <TR valign=top>\n <TD rowspan=2 width=90 height=69 valign=middle><div >    <div align=center><h1>Ethernet</h1></div> </div>  </TD>\n <TD width=45 height=69 valign=middle><div>    <div align=center><h1>COM1</h1></div> </div>  </TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL0</h1></div>  <label class='switch'>  <input type='checkbox' id='rl0' onclick='relayCheck(0)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL1</h1></div>  <label class='switch'>  <input type='checkbox' id='rl1' onclick='relayCheck(1)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL2</h1></div>  <label class='switch'>  <input type='checkbox' id='rl2' onclick='relayCheck(2)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL3</h1></div>  <label class='switch'>  <input type='checkbox' id='rl3' onclick='relayCheck(3)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL4</h1></div>  <label class='switch'>  <input type='checkbox' id='rl4' onclick='relayCheck(4)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL5</h1></div>  <label class='switch'>  <input type='checkbox' id='rl5' onclick='relayCheck(5)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL6</h1></div>  <label class='switch'>  <input type='checkbox' id='rl6' onclick='relayCheck(6)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL7</h1></div>  <label class='switch'>  <input type='checkbox' id='rl7' onclick='relayCheck(7)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL8</h1></div>  <label class='switch'>  <input type='checkbox' id='rl8' onclick='relayCheck(8)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=top><div> <div align=center><h1>RL9</h1></div>  <label class='switch'>  <input type='checkbox' id='rl9' onclick='relayCheck(9)'>  <div class='slider'></div></label></div></TD>\n <TD width=45 height=69 valign=middle><div>  <div align=center><h1>COM2</h1></div> </div>  </TD>\n <TD width=80 height=69 valign=top><div> <div align=center><h1>TR10 - PWM</h1></div>   <div align=center> <input type='text' maxlength=3 onfocus=\"this.value=''\" value='0' id='tr10' style=' width:70%; text-align:center;'> </div>\n <div align=center> <button id='tr10button' onclick='pwmSend(0)'>Send</button>  </div>  </div>  </TD>\n <TD width=80 height=69 valign=top><div> <div align=center><h1>TR11 - PWM</h1></div>   <div align=center> <input type='text' maxlength=3 onfocus=\"this.value=''\" value='0' id='tr11' style='width:70%; text-align:center;' </div>\n <div align=center> <button id='tr11button' onclick='pwmSend(1)'>Send</button>  </div>  </div>  </TD>\n <TD width=40 height=69 valign=middle><div>  <div align=center>      <h1>TR.</h1>      <h1>COM</h1>    </div>  </div>  </TD>\n <TD width=40 height=69 valign=middle><div>  <div align=center>      <h1>TR.</h1>      <h1>SUP</h1>    </div>  </div>  </TD>\n <TD rowspan=2 width=110 height=69 valign=middle><div> <div align=center>      <h1>POWER</h1>      <h1>24 VDC</h1>   </div>  </div>  </TD>\n </TR> <TR valign=top> <TD colspan=12 height=27 valign=middle><div >   <div align=middle><h1>Relay Outputs</h1></div>  </div>  </TD>\n <TD colspan=4 height=27 valign=middle><div >  <div align=middle><h1>Transistor Outputs</h1></div> </div>  </TD>\n </TR>\n </TABLE>\n </div>\n </div>\n </div>\n \n <div id='text1' style='position:absolute; overflow:hidden; left:362px; top:8px; width:320px; height:33px; z-index:2'>\n </div>\n \n </div>\n \n </head>\n \n <body onload = 'GetSwitchState()' >\n \n <script>\n var r=[], t=[], a=[], n=[];\n var ana0=0,ana1=0,ana2=0,ana3=0,pw1=0,pw2=0;\n \n function relayCheck(rel)\n {\n var names='rl', bufname='', sn='';\n for (i = 0; i < 10; i++)\n {\n buffname = names + i;\n if(document.getElementById(buffname).checked) {sn=sn+'1'; r[i]=1;} else {sn=sn+'0'; r[i]=0;} \n \n }\n reqval=parseInt(sn,2);\n typ='r';\n }\n function pwmSend(pwm)\n {\n var names='tr1'+pwm;\n reqval=document.getElementById(names).value;\n if(reqval>255){reqval=255;}else if(reqval<0){reqval=0;}\n document.getElementById(names).value=reqval;\n if(pwm==1) {typ='s'; pw2=reqval;}else{typ='t';pw1=reqval;}\n }\n function setvalues(inpts,an0,an1,an2,an3,outs,pwm1,pwm2)\n {\n var inm='in',onm='rl',buf='';\n var i = 0;\n if(ana0!=an0) {ana0=an0; document.getElementById('a0').innerHTML=an0; }\n if(ana1!=an1) {ana1=an1; document.getElementById('a1').innerHTML=an1; }\n if(ana2!=an2) {ana2=an2; document.getElementById('a2').innerHTML=an2; }\n if(ana3!=an3) {ana3=an3; document.getElementById('a3').innerHTML=an3; }\n if(pw1!=pwm1) {pw1=pwm1; document.getElementById('tr10').value=pwm1; }\n if(pw2!=pwm2) {pw2=pwm2; document.getElementById('tr11').value=pwm2; }\n for (i=0;i<14;i++)\n {\n buf=inm+i;\n if( (inpts>>i & 1)==1){document.getElementById(buf).style.backgroundColor='#2196F3';}\n else {document.getElementById(buf).style.backgroundColor='#808080';}\n }\n for (i=0;i<10;i++)\n {\n buf=onm+i;;\n if((outs>>i & 1)==1){document.getElementById(buf).checked=1;}\n else {document.getElementById(buf).checked=0;}\n }\n \n }\n function doNone()\n {\n }\n \n setvalues(16541,20,31000,200,947,5,15,23);\n </script>\n </body>\n </html>\n"};


void Init_ethernet() // Initializes the Ethernet port and connects it to the router
{
  byte ip[] = { t1, t2, t3, t4 };
  byte subnet[] = { t1s, t2s, t3s, t4s };
  byte dnsServer[] = { t1g, t2g, t3g, t4g };
  byte gateway[] = {192, 168, 1, 1};
  int just=0;
  Serial.println(ip[1]);
  Ethernet.init(48);
//  just = Ethernet.begin(mac); Serial.println("eth ..");
  delay(1000);
  #if defined(WIZ550io_WITH_MACADDRESS)
    Ethernet.begin(ip); Serial.println("eth 0");
  #else 
    Ethernet.begin(mac,ip,dnsServer,gateway,subnet); Serial.println("eth 1");
    //Ethernet.begin(mac); Serial.println("eth 1");
  #endif
  
  server.begin();
  Serial.print("server is at ");
  Serial.println(Ethernet.localIP());
}

void pageSelector()
{
  if(page == 0) { display1(); }
  else if(page == 1) { display2(); }
  else if(page == 2) { display3(); }
  else if(page == 3) { display4(); }
  else if(page == 4) { display5(); }
}

void readBut() // reads the front panel buttons
{  
  if(digitalRead(b1))
  {
    while(digitalRead(b1)) // Down
    {
      pageSelector();
    }
    but1 = 1; blinknow = 1;
  }
  else if(digitalRead(b2)) // right
  {
    while(digitalRead(b2))
    {
      pageSelector();
    }
    but2 = 1; blinknow = 1;
  }
  else if(digitalRead(b3)) // left
  {
    while(digitalRead(b3))
    {
      pageSelector();
    }
    but3 = 1; blinknow = 1;
  }
  else if(digitalRead(b4)) // up
  {
    while(digitalRead(b4))
    {
      pageSelector();
    }
    but4 = 1; blinknow = 1;
  }
}


void readInput() // Reads all 24V digital Inputs
{
  int16_t totalIn = 0;
  in[0] = !digitalRead(IN1); totalIn|=in[0]; totalIn<<=1;
  in[1] = !digitalRead(IN2); totalIn|=in[1]; totalIn<<=1;
  in[2] = !digitalRead(IN3); totalIn|=in[2]; totalIn<<=1;
  in[3] = !digitalRead(IN4); totalIn|=in[3]; totalIn<<=1;
  in[4] = !digitalRead(IN5); totalIn|=in[4]; totalIn<<=1;
  in[5] = !digitalRead(IN6); totalIn|=in[5]; totalIn<<=1;
  in[6] = !digitalRead(IN7); totalIn|=in[6]; totalIn<<=1;
  in[7] = !digitalRead(IN8); totalIn|=in[7]; totalIn<<=1;
  in[8] = !digitalRead(IN9); totalIn|=in[8]; totalIn<<=1;
  in[9] = !digitalRead(IN10); totalIn|=in[9]; totalIn<<=1;
  in[10] = !digitalRead(IN11); totalIn|=in[10]; totalIn<<=1;
  in[11] = !digitalRead(IN12); totalIn|=in[11]; totalIn<<=1;
  in[12] = !digitalRead(IN13); totalIn|=in[12]; totalIn<<=1;
  in[13] = !digitalRead(IN14); totalIn|=in[13]; totalIn<<=1;
  if(prevIn != totalIn) { valsChanged = 1; prevIn = totalIn; }
}

int16_t ads1115(byte channel){ // Used to read the analog ports. four channels are available
  int16_t aadc0 = 0;
  byte address,data1,data2,error;
  
  switch (channel){ // selects the channel, with it the address
    case 0: address = 0x42;
    break;
    case 1: address = 0x52;
    break;
    case 2: address = 0x62;
    break;
    case 3: address = 0x72;
    break;
  }
  Wire.beginTransmission(0x49); // Begins I2C transmission with device ID
  Wire.write(0x01);
  Wire.write(address);
  //Wire.write(0x03);
  Wire.write(0x83); // Initialize the device for single read action
  error = Wire.endTransmission();
  delay(2); 
  Wire.beginTransmission(0x49);
  Wire.write(0x00);
  Wire.endTransmission();
  delay(1);

  Wire.requestFrom(0x49, 2);  // Requests the analog chip to send data
  data1 = Wire.read();
  data2 = Wire.read();
  Wire.endTransmission();
  
  aadc0 = data1<<8 | data2; // combines the data together
  
  return aadc0; // A 16 BIT VALUE varying from -32768 to 32767
}

void readAnalog() // Reads all 4 analog ports
{
  an3 = ads1115(1); delay(20); //a3
  an1 = ads1115(2); delay(20); //a1
  an2 = ads1115(0); delay(20); //a2  
  an0 = ads1115(3); delay(20); //a0
  if(prevan0!=an0 ||  prevan1!=an1 ||  prevan2!=an2 ||  prevan3!=an3) // checking if any analog inputs have changed or not, if changed "valsChanged" bit would be used to send values to the web
  {
    prevan0 = an0; prevan1 = an1; prevan2 = an2; prevan3 = an3; valsChanged = 1;
  }
}

// CHAR handling methods, here "STRING" data type is not used, all data is handled as char
void StrClear(char *str, char length)
{
    for (int i = 0; i < length; i++) {
        str[i] = 0;
    }
}

char StrContains(char *str, char *sfind)
{
    char found = 0;
    char index = 0;
    char len;
    len = strlen(str);
    if (strlen(sfind) > len) {
        return 0;
    }
    while (index < len) {
        if (str[index] == sfind[found]) {
            found++;
            if (strlen(sfind) == found) {
                return 1;
            }
        }
        else {
            found = 0;
        }
        index++;
    }
    return 0;
}
void SwitchOutputs() // Used to switch On and OFF outputs
{
  int16_t totalOut=0;
  digitalWrite(O1, out[0]); bitWrite(totalOut, 0, out[0]);
  digitalWrite(O2, out[1]); bitWrite(totalOut, 1, out[1]);
  digitalWrite(O3, out[2]); bitWrite(totalOut, 2, out[2]);
  digitalWrite(O4, out[3]); bitWrite(totalOut, 3, out[3]);
  digitalWrite(O5, out[4]); bitWrite(totalOut, 4, out[4]);
  digitalWrite(O6, out[5]); bitWrite(totalOut, 5, out[5]);
  digitalWrite(O7, out[6]); bitWrite(totalOut, 6, out[6]);
  digitalWrite(O8, out[7]); bitWrite(totalOut, 7, out[7]);
  digitalWrite(O9, out[8]); bitWrite(totalOut, 8, out[8]);
  digitalWrite(O10, out[9]); bitWrite(totalOut, 9, out[9]);
  if(changeWeb) { changeWeb = 0; prevOut = totalOut; } // used to sendata to the client
  else if(prevOut!=totalOut) { valsChanged = 1; prevOut = totalOut;}
}

void sendData(EthernetClient client) // encodes all information and sends it to the client web page
{
  if(valsChanged) // if any variable changed, then encode data and send it
  {
    int16_t inps=0,outs=0;
    for(int i = 0; i<14; i++) // All inputs are being converted into one INTEGER
    {
      inps = inps<<1;
      inps |= in[i];
//      Serial.print(in[i]); Serial.print("   "); Serial.println(inps); // can check whats being encoded using the serial print
    }
    for(int i=0;i<10;i++) // All outputs are being converted into one INTEGER
    {
      outs = outs<<1;
      outs |= out[9-i];
//      Serial.println(outs); // can check whats being encoded using the serial print
    }

    // Sends the data to the web page according to the following format
    // setvalues(inps, an0, an1, an2, an3, outs, pwm1, pwm2);
    client.print("setvalues(");
    client.print(inps); client.print(",");
    client.print(an0); client.print(",");
    client.print(an1); client.print(",");
    client.print(an2); client.print(",");
    client.print(an3); client.print(",");
    client.print(outs); client.print(",");
    client.print(pwm1); client.print(",");
    client.print(pwm2); client.print(");");

    // Serial printing to double check transfer
    Serial.print("setvalues(");
    Serial.print(inps); Serial.print(",");
    Serial.print(an0); Serial.print(",");
    Serial.print(an1); Serial.print(",");
    Serial.print(an2); Serial.print(",");
    Serial.print(an3); Serial.print(",");
    Serial.print(outs); Serial.print(",");
    Serial.print(pwm1); Serial.print(",");
    Serial.print(pwm2); Serial.print(");");

    valsChanged = 0;
  }
  else // if values arent changed then the following is sent 
  {
    client.println("doNone();");
  }
}

void GetSwitchState(EthernetClient client,  int idno, int bufVal) // Handles requests from the web client
{
  if(idno == 0) // Receives relay information from the webpage
  {
    for(int i=0;i<10;i++)
    {
      out[9-i]=bitRead(bufVal, i);
    }
    changeWeb = 1;
    SwitchOutputs(); 
    sendData(client);
  }
  else if(idno == 1) // Receives transistor 1 PWM value 
  {
    pwm1 = bufVal;
    analogWrite(O11, pwm1);
    sendData(client);
  }
  else if(idno == 2) // Receives transistor 2 PWM value 
  {
    pwm2 = bufVal;
    analogWrite(O12, pwm2);
    sendData(client);
  }
  else if(idno == 3) // collects analog data and sends it to the client
  {
    readAnalog();
    readInput();
    sendData(client);
  }  
}

void ethernetCheck() // Checks if an ethernet client is available or not for connection
{
  int sps = 29;
  int sendVal=0,offsetVal=48, checkamount;
  char bufChar;
  EthernetClient client = server.available();
  if (client) { // checks if client is connected and available ornot
    Serial.println("connection made");
    boolean currentLineIsBlank = false;
    while (client.connected()) { // keeps the connection up until done
      if (client.available()) { 
        char c = client.read(); // reads the data sent from the web page
        if (req_index < (REQ_BUF_SZ - 1)) {
          HTTP_req[req_index] = c;          
          req_index++;
        }
//        Serial.print(req_index); Serial.print(" "); Serial.println(c);

//        Serial.write(c); // Use this to check what data is being sent from the web page

        if (c == '\n' && currentLineIsBlank) // if all data is sent then process it
        {
          // Html declarations
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/html");
          client.println("Connnection: close");
          client.println();

          // checking for ajax switch status and process all the information, 
          if(StrContains(HTTP_req, "ajax_switch&nocache=r")) // used to recieve relay related information
          {
            sendVal = (HTTP_req[sps] - offsetVal)*1000 + (HTTP_req[sps+1] - offsetVal)*100 + (HTTP_req[sps+2] - offsetVal)*10 + (HTTP_req[sps+3] - offsetVal);
            GetSwitchState(client, 0, sendVal);
          }
          else if(StrContains(HTTP_req, "ajax_switch&nocache=t")) // transistor 1
          {
            sendVal = (HTTP_req[sps+1] - offsetVal)*100 + (HTTP_req[sps+2] - offsetVal)*10 + (HTTP_req[sps+3] - offsetVal);
            GetSwitchState(client, 1, sendVal);
          }
          else if(StrContains(HTTP_req, "ajax_switch&nocache=s")) // transistor 2
          {
            sendVal = (HTTP_req[sps+1] - offsetVal)*100 + (HTTP_req[sps+2] - offsetVal)*10 + (HTTP_req[sps+3] - offsetVal);
            GetSwitchState(client, 2, sendVal);
          }
          else if(StrContains(HTTP_req, "ajax_switch&nocache=0")) // if nonse send an update
          {
            sendVal = 0;
            GetSwitchState(client, 3, sendVal);
          }
          else if ( StrContains(HTTP_req, "GET / ") || StrContains(HTTP_req, "GET /index.htm") )  // if no ajax switch then send the HTML page
          {
            // open web page file
            //
            Serial.println("sending page"); 
            Serial.print("page size: ");
            Serial.println(strlen_P(HtmlPage));
            for(int k = 0; k < strlen_P(HtmlPage); k++) // Read flash memory and obtain the HTML page and send it
            {
              bufChar = pgm_read_byte_near(HtmlPage + k);
              client.print(bufChar);
            }
            client.println("</script>");
            client.println("</body>");
            client.println("</html>");
            
          }
          req_index = 0;
          StrClear(HTTP_req, REQ_BUF_SZ);
          break;
        }
        if (c == '\n') {
          currentLineIsBlank = true;
        } 
        else if (c != '\r') {
          currentLineIsBlank = false;
        }
      }
    }
    delay(1);
    client.stop(); // after transactions are complelted, stop the client
  }          
}

void display1() // displays inputs status
{
  display.clearDisplay();
  display.setTextSize(0);
  display.setCursor(0,0); 
  display.print("Standard Test Program");
  display.setCursor(0,15); display.print("  Inputs Page 1");
  display.setCursor(0, 25);
  display.print("I1: "); display.print((in[0]));
  display.print(" I2: "); display.print((in[1]));
  display.print(" I3: "); display.print((in[2]));
  display.setCursor(0, 35);
  display.print("I4: "); display.print((in[3])); 
  display.print(" I5: "); display.print((in[4])); 
  display.print(" I6: "); display.print((in[5]));
  display.setCursor(0, 45);
  display.print("I7: "); display.print((in[6]));
  display.print(" I8: "); display.print((in[7]));
  display.print(" I9: "); display.print((in[8]));

  display.setCursor(0,55); display.print("Menu = Next Page");

  display.display();
}

void display2() // displays inputs status
{
  display.clearDisplay();
  display.setTextSize(0);
  display.setCursor(0,0); 
  display.print("Standard Test Program");
  display.setCursor(0,15); display.print("  Inputs Page 2");
  display.setCursor(0, 25);
  display.print("I10: "); display.print((in[9]));
  display.print(" I11: "); display.print((in[10]));
  display.print(" I12: "); display.print((in[11]));
  display.setCursor(0, 35);
  display.print("I13: "); display.print((in[12])); 
  display.print(" I14: "); display.print((in[13])); 

  //display.setCursor(0,45); display.print("IP : "); display.print(currentIP);
  
  display.setCursor(0,55); display.print("Menu = Next Page");
  
  display.display();
}

void display3() // displays analog input values
{
  display.clearDisplay();
  display.setTextSize(0);
  display.setCursor(0,0); 
  display.print("Standard Test Program");
  display.setCursor(0,15); display.print("  Analog Inputs");
  display.setCursor(0, 25);  display.print("A0: "); display.print(an0);
  display.setCursor(60, 25); display.print("A1: "); display.print(an1);
  display.setCursor(0, 35);  display.print("A2: "); display.print(an2); 
  display.setCursor(60, 35); display.print("A3: "); display.print(an3); 
  
  //display.setCursor(0,45); display.print("IP : "); display.print(currentIP);
  
  display.setCursor(0,55); display.print("Menu = Next Page");

  display.display();
}

void display4() // displays output status
{
  display.clearDisplay();
  display.setTextSize(0);
  display.setCursor(0,0); 
  display.println("Standard Test Program");
  display.setCursor(0,15); display.print("  Outputs Page 1");
  
  display.setCursor(0, 25);
  display.print("O1: "); if(selec == 0 && blinknow) { display.print("_"); } else { display.print(out[0]); }
  display.print(" O2: "); if(selec == 1 && blinknow) { display.print("_"); } else { display.print(out[1]); }
  display.print(" O3: "); if(selec == 2 && blinknow) { display.print("_"); } else { display.print(out[2]); }
  display.setCursor(0,35);
  display.print("O4: "); if(selec == 3 && blinknow) { display.print("_"); } else { display.print(out[3]); }
  display.print(" O5: "); if(selec == 4 && blinknow) { display.print("_"); } else { display.print(out[4]); }
  display.print(" O6: "); if(selec == 5 && blinknow) { display.print("_"); } else { display.print(out[5]); }
  display.setCursor(0, 45);
  display.print("O7: "); if(selec == 6 && blinknow) { display.print("_"); } else { display.print(out[6]); }
  display.print(" O8: "); if(selec == 7 && blinknow) { display.print("_"); } else { display.print(out[7]); }
  display.print(" O9: "); if(selec == 8 && blinknow) { display.print("_"); } else { display.print(out[8]); }

  display.setCursor(0,55);
  if (selec == 9) { display.print("Menu = Next Page"); } else { display.print("Menu = On Exit = Off"); }

  SwitchOutputs();

  display.display();
}

void display5() // displays transistor output status
{
  display.clearDisplay();
  display.setTextSize(0);
  display.setCursor(0,0); 
  display.println("Standard Test Program");
  display.setCursor(0,15); display.print("  Outputs Page 2");
  
  display.setCursor(0, 25);
  display.print("O10: "); if(selec == 0 && blinknow) { display.print("_"); } else { display.print(out[9]); }
  display.setCursor(0, 35);
  display.print(" O11: "); if(selec == 1 && blinknow) { display.print("_"); } else { display.print(pwm1); }
  display.setCursor(60, 35);
  display.print(" O12: "); if(selec == 2 && blinknow) { display.print("_"); } else { display.print(pwm2); }
  //display.setCursor(0,45); display.print("IP : "); display.print(currentIP);
  display.setCursor(0,55);
  if (selec == 3 && !configured) { display.print("Menu = Next Page"); } 
  else if(selec==0) { display.print("Menu = On Exit = Off"); }
  else if( (selec == 1 || selec == 2) && !configured) { display.print("Menu = Change Value"); }
  else if( (selec == 1 || selec == 2) && configured) { display.print("Menu = Confirm Val"); }

  SwitchOutputs();
  if(prevPWM1 != pwm1) { analogWrite(O11, pwm1); prevPWM1 = pwm1; valsChanged = 1; }
  if(prevPWM2 != pwm2) { analogWrite(O12, pwm1); prevPWM2 = pwm2; valsChanged = 1; }

  display.display();
}

void runCheck() // main program 
{
  readAnalog();
  Serial.println("P1a ");
  readBut();
  Serial.println("P2 ");
  readInput();
  Serial.println("P3 ");
  if(page == 3)
  {
    readBut();
    if(!but1 && !but2 && !but3 && but4) // up
    {
      selec++; 
      if(selec > 9) { selec = 0; }
      but4 = 0;
    }
    else if(but1 && !but2 && !but3 && !but4) // down
    {
      selec--;
      if(selec < 0) { selec = 9; }
      but1 = 0;
    }
    else if(!but1 && but2 && !but3 && !but4) // right
    {
      out[selec] = 0;
      but2 = 0;
    }
    else if(!but1 && !but2 && but3 && !but4) // left
    {
      if(selec == 9) { page = 4; selec = 0; }
      else
      {
        out[selec] = 1;
      }
      but3 = 0;
    }
    //confirmBut();
  }
  else if(page == 4) {
    readBut();
    if(!but1 && !but2 && !but3 && but4) // up
    {
      if(!configured){selec++;  if(selec > 3) { selec = 0; } }
      else
      {
        if(selec == 1) { pwm1++; } else if(selec == 2) { pwm2++; }
      }
      but4 = 0;
    }
    else if(but1 && !but2 && !but3 && !but4) // down
    {
      if(!configured){ selec--; if(selec < 0) { selec = 3; } }
      else
      {
        if(selec == 1) { pwm1--; } else if(selec == 2) { pwm2--; }
      }
      but1 = 0;
    }
    else if(!but1 && but2 && !but3 && !but4) // right
    {
      out[selec + 9] = 0;
      but2 = 0;
    }
    else if(!but1 && !but2 && but3 && !but4) // left
    {
      if(selec == 3) { page = 0; selec = 0; }
      else if(selec == 1 || selec == 2) 
      { 
        if(configured) { configured = 0; } else { configured = 1; }
      }
      else
      {
        out[selec + 9] = 1;
      }
      but3 = 0;
    }
    //confirmBut();
  }
  else
  {
    readInput();
    readBut();
    if(!but1 && !but2 && but3 && !but4) // left
    {
      page++; selec = 0;
      but3 = 0;
    }
    else
    {
      but1 = 0; but2 = 0; but4 = 0;
    }
    //confirmBut();
  }
  //confirmBut();

  pageSelector();
}

void setup() 
{
  //Inputs
  pinMode(IN1, INPUT);  pinMode(IN2, INPUT);  pinMode(IN3, INPUT);  pinMode(IN4, INPUT);
  pinMode(IN5, INPUT);  pinMode(IN6, INPUT);  pinMode(IN7, INPUT);  pinMode(IN8, INPUT);
  pinMode(IN9, INPUT);    pinMode(IN10, INPUT);    pinMode(IN11, INPUT);    pinMode(IN12, INPUT);
  pinMode(IN13, INPUT);   pinMode(IN14, INPUT); 
  //Relay
  pinMode(O1, OUTPUT);  pinMode(O2, OUTPUT);  pinMode(O3, OUTPUT);  pinMode(O4, OUTPUT);
  pinMode(O5, OUTPUT);  pinMode(O6, OUTPUT);  pinMode(O7, OUTPUT);  pinMode(O8, OUTPUT);
  pinMode(O9, OUTPUT);  pinMode(O10, OUTPUT);  
  // Transistors
  pinMode(O11, OUTPUT);  pinMode(O12, OUTPUT);

  digitalWrite(O1, LOW);  digitalWrite(O2, LOW);  digitalWrite(O3, LOW);  digitalWrite(O4, LOW);
  digitalWrite(O5, LOW);  digitalWrite(O6, LOW);  digitalWrite(O7, LOW);  digitalWrite(O8, LOW);
  digitalWrite(O9, LOW);  digitalWrite(O10, LOW);  
  //Transistors
  digitalWrite(O11, LOW);  digitalWrite(O12, LOW);

  pinMode(b1, INPUT); pinMode(b2, INPUT); pinMode(b3, INPUT); pinMode(b4, INPUT);

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);  // initialize with the I2C addr 0x3D (for the 128x64)
  display.clearDisplay();
  // dispaly logo
  display.setTextSize(2);
  display.setTextColor(WHITE);
  display.setCursor(0,1);
  display.print("ICONIC");
  display.setCursor(0,25);
  display.setTextSize(1);
  display.println("NORVI CONTROLLER");
  display.setCursor(0,35); display.println("Standard V4.0");
  display.setCursor(0,45); display.println("TEST PROGRAM");
  display.setCursor(0,55); display.println("14 DI / 10 RL / 2 TR");
  display.display();
  // Clear the buffer.
  display.clearDisplay();
  display.setTextSize(0);

  digitalWrite(O1, LOW);  digitalWrite(O2, LOW);  digitalWrite(O3, LOW);  digitalWrite(O4, LOW);
  digitalWrite(O5, LOW);  digitalWrite(O6, LOW);  digitalWrite(O7, LOW);  digitalWrite(O8, LOW);
  digitalWrite(O9, LOW);  digitalWrite(O10, LOW);  
  //Transistors
  digitalWrite(O11, LOW);  digitalWrite(O12, LOW);
  
  Serial.begin(9600);
  Serial.println("P1 ");
  delay(2000);
  
  Init_ethernet();

/*
  delay(2000);

  Serial.print("\nInitializing SD card...");
  if (!card.init(SPI_HALF_SPEED, SD_chipSelect)) {   Serial.println(" CARD NOT FOUND");  }
  else { Serial.println(" SD WORKING"); }
  Serial.print("\nCard type: ");
  Serial.println(card.type());
  if (!volume.init(card)) {
    Serial.println("Could not find FAT16/FAT32 partition.\nMake sure you've formatted the card");
    return;
  }
  if (!SD.begin(SD_chipSelect)) {
    Serial.println("Card failed, or not present");
    // don't do anything more:
    return;
  } 
  Serial.println("card initialized.");
*/


  //set timer3 interrupt at 1Hz
  TCCR3A = 0;// set entire TCCR1A register to 0
  TCCR3B = 0;// same for TCCR1B
  TCNT3  = 0;//initialize counter value to 0
  // set compare match register for 1hz increments
  OCR3A = 15624;// = (16*10^6) / (1*1024) - 1 (must be <65536)
  // turn on CTC mode
  TCCR3B |= (1 << WGM12);
  // Set CS12 and CS10 bits for 1024 prescaler
  TCCR3B |= (1 << CS12) | (1 << CS10);  
  // enable timer compare interrupt
  TIMSK3 |= (1 << OCIE3A);

}

void loop() 
{
  runCheck();
  ethernetCheck();
  delay(200);
}

ISR(TIMER3_COMPA_vect){
  blinknow = blinknow^1;
}
