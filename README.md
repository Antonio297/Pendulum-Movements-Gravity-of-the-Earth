//ººººººººººººººººººººººººººººººººººººººººººººººººººººººººººº
// Program_Mass_of_the_Earth
// March 21, 2022
// Creators: Noguerón, Antonio and Javier, Francisco
// Contact Noguerón, Antonio: nogueron.antonio29@gmail.com
// Contact Javier, Francisco: a20182418@pucp.edu.pe
//ººººººººººººººººººººººººººººººººººººººººººººººººººººººººººº
// The purpose of this program is to be able to obtain gravity as a variable through pendular movements. 
// These will be captured by a photoresistor that will collect the data to give us an average value.

#include <LowPower.h>
#include<Wire.h>
#include <LiquidCrystal_I2C.h>
#include<Keypad.h>

// To develop the different parts of the hardware, 
// some libraries will be needed and the Arduino IDE program will be used,
// although this may be different from another.
LiquidCrystal_I2C lcd (0x27,16,2);
byte m=0;
char Data[7];
byte cara[8]={
  B00000,
  B10001,
  B00000,
  B00000,
  B10001,
  B01110,
  B00000,
};

// Later we have to define certain variables that will allow us to enter the necessary 
// values, such as our length of the respective pendulum and the options when taking the data.
String msg1="-Ingresa la longitud-";
String msg2= "Tú longitud es:";
float longitud;
int pinBoton=10;
const int pinLASER=11;
float tiempo=0;
float tiempo1=0;
float tiempocorte1=0;
float tiempocorte2=0;
float valorCorte2;
float valorCorte3;
int i=0;
int c=0;
const byte filas = 4;
const byte columnas = 4;
byte pinsFilas[filas] = {9, 8, 7, 6};
byte pinsColumnas[columnas] = {5, 4, 3, 2};

// For this work, it was developed in conjunction with a 12C module. 
// Also as an addition, a character was included that will give the shape of
// a smile when entering the program.


char keys[filas][columnas] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','.'}// in this case, the letter D will work as a period.
 };

// As an additional, if we want to have a matrix keyboard, 
// it will be essential to define the number of rows and columns as a matrix.

Keypad teclado = Keypad(makeKeymap(keys), pinsFilas, pinsColumnas, filas, columnas);

float voltage=0;
float voltageAnterior=0;
float valorCorte=0;
int contar=0;
boolean estado=false;
boolean estadoBoton=LOW;
String mensaje="Por favor, ingresa la longitud";
String mensaje1="Ingresa la ";
String mensaje2="-longitud-";

void setup(){
  lcd.init();
  lcd.backlight();
  lcd.createChar(6,cara);
  lcd.clear();
  lcd.setCursor(3,0);
  lcd.print("Bienvenido");
  lcd.setCursor(6,1);
  lcd.write(byte(6));
  lcd.setCursor(10,1);
  lcd.write(byte(6));
  delay(4000);
  lcd.clear();
  
// Next we have to configure the Void setup, starting the LCD screen with a welcome message.
// Keep in mind that these factors are totally optional, however they could give
// a better experience when taking our data.
  pinMode(pinBoton,INPUT);
  pinMode(pinLASER,OUTPUT);
  Serial.begin(9600);
  digitalWrite(pinLASER,LOW); // The laser starts, logically, off
  Serial.println();
  Serial.println("Bienvenido");
  Serial.println(msg1);
  delay(2000);
  lcd.clear();
  lcd.begin(16, 2);      // Initialize the screen
  lcd.setCursor(3, 0);   // Position the cursor at position (3,0)
  lcd.print(mensaje1);   // Show the message
  lcd.setCursor(3, 1);   // Position the cursor at position (3,1)
  lcd.print(mensaje2);   // Show the message
  delay(4000);
  lcd.clear();
  Serial.println("Longitud: ");
  
  
}

void loop(){

// But to define the length, we would have a problem with the matrix keyboard.
// It does not allow the entry of the variable as a numerical value. 
// For this, the entered numbers are stored in an array.
  lcd.setCursor(0,0);
  lcd.print("Longitud:");
  char tecla=teclado.getKey();
  if(tecla){
    Data[m]=tecla;
    Serial.print(tecla);
    lcd.setCursor(m,1);
    lcd.print(Data[m]);
    m++;
    while(tecla=='A'){
      float cn= (Data[0]-'0')*10;
      float dc= (Data[1]-'0')*1;
      float pd= (Data[2]-'0')*0.1;
      float sd= (Data[3]-'0')*0.01;
      float td= (Data[4]-'0')*0.001;
      float ct= (Data[5]-'0')*0.0001;
      longitud= cn+dc+pd+sd+td+ct;
      temp();
    
    }
  }
}
  // Serial.print(msg2);
  // Serial.println(String(longitud)+"cm");
void temp(){

// Subsequently, the temp function will take care of giving us a signal with the led,
// to know that the value was entered successfully. 
// And also, it will allow us to turn it on so that data collection begins.
  
  if(valorCorte<=0){
    vcorte();  //If the cutoff value is not set, sets it
  }
  int estadoBotonAhora=digitalRead(pinBoton);
  if(estadoBotonAhora==HIGH && estadoBoton==LOW){
     estado= estado xor 1;
     delay(50);
  }
  estadoBoton= estadoBotonAhora;
  digitalWrite(pinLASER,estado);
  // End of turning on the laser
  
  tiempo=millis();
  int sensorValue=analogRead(A0);
  float voltage=sensorValue;
  String voltaje=String(voltage);
  voltaje.replace(".",",");
  //Serial.println(String(tiempo)+";"+voltaje);
  delay(10);
  //Reconocimiento del paso
  if(estado==HIGH && voltage<valorCorte && voltageAnterior>=valorCorte){
    contar=contar+1;
    if(contar==1){
      tiempocorte1=tiempo;
      Serial.println("===========================");
      Serial.println("Valor de corte:"+ String(valorCorte));
      Serial.println("Primer paso en el valor de tiempo(ms):"+String(float(tiempocorte1)));
      lcd.setCursor(0,0);
      lcd.print("Recopilando");
      lcd.setCursor(0,1);
      lcd.print("datos: " +String(longitud/100)+ "m");
      Serial.println(voltage);
    }else if(contar==3 && tiempocorte1>0){
      double diferencia=((tiempo-tiempocorte1)/1000);
      double diferencia2=(tiempo-tiempocorte1);
      double gravedad=(39.48*(longitud/100)/(diferencia*diferencia));
      double periodo=((diferencia2)/1000);
      float gravity[10];
      float period[10];
      i++;
      c++;
      gravity[i]=gravedad;
      period[c]=periodo;
      float grav=(gravity[0]+gravity[1]+gravity[2]+gravity[3]+gravity[4]+gravity[5]+gravity[6]+gravity[7]+gravity[8]+gravity[9]+gravity[10])/10; 
      float per=(period[0]+period[1]+period[2]+period[3]+period[4]+period[5]+period[6]+period[7]+period[8]+period[9]+period[10])/10;
      if(i==10){
      Serial.println("g="+String(grav,4)+"m/s^2");
      Serial.println("T="+String(per,4)+"s");
      Serial.println("L= "+String(longitud/100)+"m");
      delay(4000);
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("T:"+String(per,4)+"s");
      lcd.setCursor(0,1);
      lcd.print("g:"+String(grav,4)+"m/s^2");
      delay(15000);
      lcd.clear();
      lcd.setCursor(2,0);
      lcd.print("-Presiona B-");
      lcd.setCursor(0,1);
      lcd.print("para tomar datos");
      Serial.println("===========================");
      Serial.println("-Presiona B-");
      Serial.println("para tomar datos");
      delay(5000);
      lcd.clear();
      lcd.setCursor(2,0);
      lcd.print("-Presiona C-");
      lcd.setCursor(1,1);
      lcd.print("para finalizar");
      Serial.println("-Presiona C-");
      Serial.println("para finalizar");
      Serial.println("===========================");
      delay(5000);
      lcd.clear();
      goto out;
      } 
      Serial.println("T="+String(float(diferencia2)/1000)+"s");
      Serial.println("g="+String(float((38.48*(longitud/100))/(diferencia*diferencia)))+"m/s^2");
      Serial.println("=========================");
      contar=0;
      tiempocorte1=0;
      }
    }
 voltageAnterior=voltage;

 out:
 
 void(* resetFunc) (void) = 0; 
   char tecla=teclado.getKey();
    if(tecla=='B'){
      pinMode(pinBoton,INPUT);
      pinMode(pinLASER,OUTPUT);
      Serial.begin(9600);
      digitalWrite(pinLASER,LOW); //El laser empieza, lógicamente, apagado
      LowPower.powerDown(SLEEP_4S, ADC_OFF, BOD_OFF);
      resetFunc();
      setup();
    }
    if(tecla=='C'){
      
      pinMode(pinBoton,INPUT);
      pinMode(pinLASER,OUTPUT);
      Serial.begin(9600);
      digitalWrite(pinLASER,LOW);
      lcd.setCursor(3,0);
      lcd.print("-Gracias-");
      lcd.setCursor(1,1);
      lcd.print("vuelva pronto.");
      Serial.println("-Gracias-");
      Serial.println("vuelva pronto.");
      Serial.println("=========================");
      delay(10000);
      lcd.clear();
      LowPower.powerDown(SLEEP_FOREVER, ADC_OFF, BOD_OFF);
      }
    
  if (Serial.available()>0){
    //leemos la opcion enviada
    int opcion=Serial.read();
    if(opcion=='B') {
      pinMode(pinBoton,INPUT);
      pinMode(pinLASER,OUTPUT);
      Serial.begin(9600);
      digitalWrite(pinLASER,LOW); //El laser empieza, lógicamente, apagado
      LowPower.powerDown(SLEEP_4S, ADC_OFF, BOD_OFF);
      resetFunc();
      setup();
    }
    if(opcion=='C') {
      pinMode(pinBoton,INPUT);
      pinMode(pinLASER,OUTPUT);
      Serial.begin(9600);
      digitalWrite(pinLASER,LOW);
      lcd.setCursor(3,0);
      lcd.print("-Gracias- ");
      lcd.setCursor(1,1);
      lcd.print("vuelva pronto.");
      Serial.println("-Gracias-");
      Serial.println("vuelva pronto.");
      Serial.println("=========================");
      delay(10000);
      lcd.clear();
      lcd.backlight();
      LowPower.powerDown(SLEEP_FOREVER, ADC_OFF, BOD_OFF);
      
     }
   }
 }

void vcorte(){
   digitalWrite(pinLASER,LOW);
   valorCorte2=analogRead(A0);
   delay(10);
   digitalWrite(pinLASER,HIGH);
   valorCorte3=analogRead(A0);
   delay(10);
   valorCorte=(valorCorte2+valorCorte3)/2; //La semisuma
   digitalWrite(pinLASER,estado);
}
