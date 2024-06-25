#include <Wire.h>
#include <Keypad.h>
#include <LiquidCrystal_I2C.h>
#include <EEPROM.h>

// --- pin ---

char pin[4] = "1234";
void PINActual (char* pin){
  EEPROM.put(0,pin);
}

//  --- Leds ---
int led1 = 3;
int led2 = 5;
int led3 = A2;

// --- Booleanos Menu ---
bool mPrincipal = true;
bool mAlarma = false;
bool mAlarma2 = false;
bool mAlarma3 = false;
bool mSensores = false;
bool mConfig = false;
bool menuPIN = false;
bool mAlarmaEstado = false;
bool mNuevoPIN = false;
bool mEstadoAlarma = false;
bool mIngresarEstadoAlarma = false;
// ---
long duracion, cm,duracion2,cm2,duracion3,cm3;
// --- Sensores y Alarma ---
const int pingPin = A1; // Entrada 
const int pingPin2 = 1; // Cocina
const int pingPin3 = 2; // Habitacion

const int alarma = 13;

bool sensorActivado = false;
bool sensorActivado2 = false;
bool sensorActivado3 = false;
// --- TECLADO ---
const byte FILAS = 4;
const byte COLS = 4;
char teclas[FILAS][COLS]=
{
	'1','2','3','A',
  	'4','5','6','B',
  	'7','8','9','C',
  	'*','0','#','D'
};
byte pinesFilas[FILAS] ={12,11,10,9};
byte pinesCols[COLS] = {8,7,6,4};
Keypad kp = Keypad(makeKeymap(teclas), pinesFilas, pinesCols, FILAS, COLS);
// --- LCD ---
const byte MAX_CHARS = 16;
const byte MAX_ROWS = 2;
const byte SP_CHAR = 8;
LiquidCrystal_I2C lcd(0x20, MAX_CHARS, MAX_ROWS);
// --- Menu Principal ---
void menuPrincipal (){
  lcd.setCursor(0,0);
  lcd.print("1.Config/2.Alarm");
  lcd.setCursor(0,1);
  lcd.print("3. Eventos");
  char pinActual[] = "    ";
  
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    switch (tecla) {
      case '1':
      	mPrincipal = false;
        lcd.clear();
      	mConfig = true;
      break;
      case '2':
      	mPrincipal = false;
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Ingrese PIN: ");
        lcd.setCursor(0,1);
      	mIngresarEstadoAlarma = true;
      break;
      case '3':
      	//mPrincipal = false;
        //lcd.clear();
      break;
    } 
  }  
}
// --- Menu Configuracion ---
void menuConfiguracion (){
  lcd.setCursor(0,0);
  lcd.print("1.PIN/2.Sensores");
  lcd.setCursor(0,1);
  lcd.print("3.Modo Alarma");
  
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    switch (tecla) {
      case '1':
      	mConfig = false;  
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Ingrese PIN: ");
        lcd.setCursor(0,1);
      	menuPIN = true;
      break;
      case '2':
      	mConfig = false;
      	lcd.clear();
      	mSensores = true;
      break;/*
      case '3':
      	mConfig = false;
      	mAlarmaEstado = true;
      break;*/
      case 'A':
      	mConfig = false;
        lcd.clear();
      	mPrincipal = true;
      break;
      case 'C':
      	mConfig = false;
        lcd.clear();
      	mPrincipal = true;
      break;
    } 
  }
}
int i = 0;
bool pinIgual = true;
void mPIN(){
  
  //PINActual();
  char pinActual[4] = "";
  char actual[4]="";
  pinActual[0] = EEPROM.read(0);
  pinActual[1] = EEPROM.read(1);
  pinActual[2] = EEPROM.read(2);
  pinActual[3] = EEPROM.read(3);
  

  char tecla = kp.getKey();
  if( i < 4){
    if (tecla != NO_KEY){
      switch (tecla) {
        case '#': 
        	i=4;
        break;
        case 'D':
          lcd.clear();  
          lcd.setCursor(0,0);
          lcd.print("Ingrese PIN: ");
          lcd.setCursor(0,1);
          i = 0;
        break;
        case 'A':
        	menuPIN = false;
        	lcd.clear();
        	mConfig = true;
        break;
        case 'C':
        	menuPIN = false;
        	lcd.clear();
        	mPrincipal = true;
        default:
        if(tecla >= '0' && tecla <='9'){
          lcd.print("*");
          actual[i] = tecla;
          
          i++;
        }
        break;
      }
    }
  }
  if(i = 4 && tecla == '#'){    
    if(strcmp(pinActual, actual)){
      pinIgual = true;
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Pin Correcto.");
      delay(100);
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Ing. nuevo PIN:");
      lcd.setCursor(0,1);
      
      menuPIN = false;
      mNuevoPIN =true;

      i = 0;

    }else{
      pinIgual = false;
      Serial.println(pinActual);
      Serial.println(actual);
    }
    if(!pinIgual){
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Pin Incorrecto.");
      
      strcpy( actual, "");
      i = 0;
      delay(100);
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Ingrese PIN:");
      lcd.setCursor(0,1);
      
    }
  }
}
int j=0;
void nuevoPIN () {
  
  char nuevo[4] = "";
  nuevo[0] = EEPROM.read(0);
  nuevo[1] = EEPROM.read(1);
  nuevo[2] = EEPROM.read(2);
  nuevo[3] = EEPROM.read(3);
  
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    if(j < 4){
      if(tecla == 'D'){
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Ingrese nuevo PIN:");
        lcd.setCursor(0,1);
      }
      if(tecla >= '0' && tecla <= '9'){
        nuevo[j] = tecla;
        lcd.print(tecla);
        j++;
      }

    }
    if(j = 4 && tecla == '#'){

      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("PIN guardado.");
      PINActual(nuevo);
      delay(100);
      Serial.print(nuevo);
      menuPIN = false;
      nuevo[0] = EEPROM.read(0);
      nuevo[1] = EEPROM.read(1);
      nuevo[2] = EEPROM.read(2);
      nuevo[3] = EEPROM.read(3);
      Serial.println(nuevo);
      lcd.clear();
      lcd.setCursor(0,0);
      mNuevoPIN = false;
      lcd.clear();
      mConfig = true;
    }
  }  
}
// --- Menu Sensores ---
void menuSensores(){
  lcd.setCursor(0,0);
  lcd.print("1.Hab/2.Cocina");
  lcd.setCursor(0,1);
  lcd.print("3. Entrada");
  
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    switch (tecla) {
      case '1':
      	mSensores = false;
      	lcd.clear();
      	mAlarma = true;
      break;
      case '2':
      	mSensores = false;
      	lcd.clear();
      	mAlarma2 = true;
      break;
      case '3':
      	mSensores = false;
      	lcd.clear();
      	mAlarma3 = true;
      break;
        case 'A':
        	mSensores = false;
        	lcd.clear();
        	mConfig = true;
        break;
        case 'C':
        	mSensores = false;
        	lcd.clear();
        	mPrincipal = true;
        break;
    } 
  }
  
}
// --- Estado Alarma ---
int k=0;
char comprobar[4] = "";
void ingresarEstadoAlarma(){
  char pin[4] = "";
  pin [0] = EEPROM.read(0);
  pin [1] = EEPROM.read(1);
  pin [2] = EEPROM.read(2);
  pin [3] = EEPROM.read(3);

  char tecla = kp.getKey();
  if( k < 4){
    if (tecla != NO_KEY){
      switch (tecla) {
        case '#': 
        	k=4;
        break;
        case 'D':
        	lcd.clear();
          lcd.clear();  
          lcd.setCursor(0,0);
          lcd.print("Ingrese PIN: ");
          lcd.setCursor(0,1);
          k = 0;
        break;
        case 'A':
        	mIngresarEstadoAlarma = false;
        	lcd.clear();
        	mPrincipal = true;
        break;
        case 'C':
        	mIngresarEstadoAlarma = false;
        	lcd.clear();
        	mPrincipal = true;
        default:
        if(tecla >= '0' && tecla <='9'){
          lcd.print("*");
          comprobar[k] = tecla;
          k++;
        }
        break;
      }
    }
  }
  if(k = 4 && tecla == '#'){    
    if(strcmp(pin, comprobar)){
      pinIgual = true;
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Pin Correcto.");
      delay(100);
      mIngresarEstadoAlarma = false;
      lcd.clear();
      mEstadoAlarma =true;

      lcd.clear();
          k = 0;
    }
  }
}
void estadoAlarma(){
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    switch (tecla) {
      case '1':
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Alarma Activada.");
        pinMode(alarma,HIGH);
        delay(100);
        lcd.clear();
        mEstadoAlarma = false;
      	mPrincipal = true;
      break;
      case '2':
        pinMode(alarma,LOW);
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Alarma Desctiv.");
        delay(100);
        lcd.clear();
        mEstadoAlarma = false;
      	mPrincipal = true;
      break;
      case 'C':
        mEstadoAlarma = false;
        lcd.clear();
      	mPrincipal = true;
      break;
      case 'A':
        mEstadoAlarma = false;
        lcd.clear();
      	mPrincipal = true;
      break;
    } 
  }
}
// --- SETUP ---
void setup() {
  
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
  //lcd.blink();
  
  pinMode (pingPin, OUTPUT);
  pinMode (pingPin2, OUTPUT);
  pinMode (pingPin3, OUTPUT);
  
  pinMode (led1, OUTPUT);
  pinMode (led2, OUTPUT);
  pinMode (led3, OUTPUT);
  analogWrite (led1, 128);
  analogWrite (led2, 128);
  analogWrite (led3, 128);
  pinMode (alarma, OUTPUT);
    
  PINActual(pin);

}
// --- LOOP ---
void loop() {
  if(mPrincipal){
    menuPrincipal();
  }else if(mAlarma){
    activar();
  }else if(mAlarma2){
    activar2();
  }else if(mAlarma3){
    activar3();
  }else if(mSensores){
    menuSensores();
  }else if(mConfig){
    menuConfiguracion();
  }else if(menuPIN){
    mPIN();
  }else if(mNuevoPIN){
    nuevoPIN();
  }else if(mEstadoAlarma){
    lcd.setCursor(0,0);
    lcd.print("Desea Activar la");
    lcd.setCursor(0,1);
    lcd.print("Alarma?1.SI/2.NO");
    estadoAlarma();
  }else if(mIngresarEstadoAlarma){
    ingresarEstadoAlarma();
  }
}
// --- Sensores ---
void Sensor(){
  
  pinMode(pingPin, OUTPUT);
  digitalWrite(pingPin, LOW);
  delayMicroseconds(2);
  digitalWrite(pingPin, HIGH);
  delayMicroseconds(5);
  digitalWrite(pingPin, LOW);
  pinMode(pingPin, INPUT);
  
}
void Sensor2(){
  pinMode(pingPin2, OUTPUT);
  digitalWrite(pingPin2, LOW);
  delayMicroseconds(2);
  digitalWrite(pingPin2, HIGH);
  delayMicroseconds(5);
  digitalWrite(pingPin2, LOW);
  pinMode(pingPin2, INPUT);
}
void Sensor3(){
  pinMode(pingPin3, OUTPUT);
  digitalWrite(pingPin3, LOW);
  delayMicroseconds(2);
  digitalWrite(pingPin3, HIGH);
  delayMicroseconds(5);
  digitalWrite(pingPin3, LOW);
  pinMode(pingPin3, INPUT);
  
}
// --- Activacion de Alarma ---
void activar(){
  lcd.setCursor(0,0);
  lcd.print("Activar Alarma?");
  lcd.setCursor(0,1);
  lcd.print("1. Si / 2. No");
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    switch (tecla) {
      case '1':
      	sensorActivado = true;
      	sensoresActivados();
        mAlarma = false;
        lcd.clear();
      	mConfig = true;
      break;
      case '2':
      	sensorActivado = false;
      	sensoresActivados();
        mAlarma = false;
        lcd.clear();
      	mConfig = true;
      break;
      case 'C':
        mAlarma = false;
        lcd.clear();
      	mPrincipal = true;
      break;
      case 'A':
        mAlarma = false;
        lcd.clear();
      	mConfig = true;
      break;
    }
  }
}
void activar2() {
  lcd.setCursor(0,0);
  lcd.print("Activar Alarma?");
  lcd.setCursor(0,1);
  lcd.print("1. Si / 2. No");
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    switch (tecla) {
      case '1':
      	sensorActivado2 = true;
      	sensoresActivados();
        mAlarma2 = false;
        lcd.clear();
      	mConfig = true;
      break;
      case '2':
      	sensorActivado2 = false;
      	sensoresActivados();
        mAlarma2 = false;
        lcd.clear();
      	mConfig = true;
      break;
      case 'C':
        mAlarma2 = false;
        lcd.clear();
      	mPrincipal = true;
      break;
      case 'A':
        mAlarma2 = false;
        lcd.clear();
      	mConfig = true;
      break;
    }
  }
}
void activar3() {
  lcd.setCursor(0,0);
  lcd.print("Activar Alarma?");
  lcd.setCursor(0,1);
  lcd.print("1. Si / 2. No");
  char tecla = kp.getKey();
  if (tecla != NO_KEY){
    switch (tecla) {
      case '1':
      	sensorActivado3 = true;
      	sensoresActivados();
        mAlarma3 = false;
        lcd.clear();
      	mConfig = true;
      break;
      case '2':
      	sensorActivado3 = false;
      	sensoresActivados();
        mAlarma3 = false;
        lcd.clear();
      	mConfig = true;
      break;
      case 'C':
        mAlarma3 = false;
        lcd.clear();
      	mPrincipal = true;
      break;
      case 'A':
        mAlarma3 = false;
        lcd.clear();
      	mConfig = true;
      break;
    }
  }
}
void sensoresActivados(){
  if(sensorActivado2){
    Sensor2();
    duracion2 = pulseIn(pingPin2, HIGH);
    cm2 = duracion2 / 29 / 2;
    Alarma(cm2);
        
  }
  if(sensorActivado){
    Sensor();
    duracion = pulseIn(pingPin, HIGH);
    cm = duracion / 29 / 2;
    Alarma(cm);
  }
  if(sensorActivado3){
    Sensor3();
    duracion3 = pulseIn(pingPin, HIGH);
    cm3 = duracion3 / 29 / 2;
    Alarma(cm3);
  }
  if(!sensorActivado && !sensorActivado2 && !sensorActivado3){ noTone(alarma);  }
}
void Alarma(long cm){
    if(cm < 50){tone(alarma,50); }	 
    else { noTone (alarma);  }
}
