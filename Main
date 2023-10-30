#include <SPIFFS.h>

#define DE_RE_PIN 4
#define MODE_SEND HIGH
#define MODE_RECV LOW

byte i = 0;
int  dataCount = 0;
float Measure[7]={};
int BanderaOk = 0; 
// Vector para almacenar los datos recibidos
byte receivedData[100]; // Ajusta el tamaño según sea necesario
uint8_t buff[] = {
    // Dirección del dispositivo
    0x01, 0x03, 0x00, 0x00, 0x00, 0x07, 0x04, 0x08  // CRC alto
};

uint8_t buffRes[] = {
    // Respueta del sensor 3 al inicio y 2 al final 
    0x01, 0x03, 0x0E// Address, Funtion code, number of byte......., Error check lo, Error check hi
};


//tiempo
unsigned long previousMillis = 0;
const long interval =  180000;//3 * 60 * 1000;  // 3 minutos en milisegundos


void readFromSerial(HardwareSerial& port) {

  digitalWrite(DE_RE_PIN, MODE_SEND);

  port.write(buff, sizeof(buff));
  port.flush();

  digitalWrite(DE_RE_PIN, MODE_RECV);
  delay(500);


  while (port.available()) {
    uint8_t data = port.read();
    // Almacena los datos en el vector
    receivedData[dataCount] = data;
    dataCount++;
  }
  
  Serial.println("-------------------");
  
  int CountRes = 0;
  uint8_t posiciones[3] = {};
  for(int i= 0; i<= dataCount ; i++ ){
    if (buffRes[CountRes] == receivedData[i]){
      Serial.println(" Posicion "+String(i));
      posiciones[CountRes] = i;
      CountRes++;
      if (CountRes == 3){
        i = dataCount;
      }
    }
  }

  if((posiciones[0])+1 == posiciones[1]){ //se pregunta si cimple con la secuencia principal
  
    if((posiciones[1])+1 == posiciones[2]){
      Serial.println(" Cumple con la Secuencia");
      BanderaOk = 1;
      int contador = 0;
      for (int i = posiciones[2]+1 ; i <= posiciones[2]+14 ; i++){
        receivedData[contador] = receivedData[i];
        contador++;
      }
    }
  }else{
      Serial.println("No Cumple con la Secuencia");
      BanderaOk = 0;
    }
  for (int i = 0; i < 14; i++) {
    Serial.print(receivedData[i], HEX);
    Serial.print(" ");
  }
  Serial.println();
}

void printAndClearData() {
  
  int CoutMeasure = 0;
  for (int i= 0; i < 14; i += 2) { // Omitir las cuatro primeras posiciones y la última
    int value1 = receivedData[i];
    int value2 = receivedData[i + 1];
    // Convierte de hexadecimal a decimal y divide entre 10
    float decimalValue = (value1 * 256 + value2) / 10.0;
    
    // Muestra el valor decimal
    //Serial.print(decimalValue, 1); // El "1" especifica 1 decimal en la salida
    //Serial.print(" ");
    Measure[CoutMeasure]=decimalValue;
    CoutMeasure ++;
  }
  for (int i = 0 ; i< 7 ; i++){
    Serial.print(Measure[i]);
    Serial.print(" ");
  }
  Serial.println();
  Serial.println("-------------------");
  
  // Limpia el vector receivedData después de imprimir los datos
  memset(receivedData, 0, sizeof(receivedData));
  dataCount = 0;
}

void setup() {
  pinMode(DE_RE_PIN, OUTPUT);
  digitalWrite(DE_RE_PIN, MODE_RECV);
  Serial.begin(115200);
  Serial2.begin(4800, SERIAL_8N1, 17, 16);
  Serial2.setTimeout(1000);
  Serial1.begin(4800, SERIAL_8N1, 14, 13); // Configura Serial1 en pines 14 (Rx) y 13 (Tx)

  if (!SPIFFS.begin()) {
    Serial.println("Error al montar el sistema de archivos SPIFFS.");
    return;
  }
}

void loop() {

  processSerialCommands();
  unsigned long currentMillis = millis();
  
  if (currentMillis - previousMillis >= interval) {
    readFromSerial(Serial2); // Lee desde Serial2
    // Realiza las operaciones necesarias con los datos leídos desde Serial2
    if (BanderaOk == 1){
      printAndClearData();
      

      File file1 = SPIFFS.open("/soil1.txt", "a");
      if (!file1) {
        Serial.println("Error al abrir el archivo soil1.txt");
      } else {
        // Escribir el valor de soil1 en el archivo y agregar una nueva línea
        for (int i = 0 ; i< 7 ; i++){
          file1.print(Measure[i]);
          file1.print(";");
        }
        file1.println();
        file1.close();
      }
    }

    readFromSerial(Serial1); // Lee desde Serial1 
    if (BanderaOk == 1){
      
      // Realiza las operaciones necesarias con los datos leídos desde Serial1
      printAndClearData();
      File file2 = SPIFFS.open("/soil2.txt", "a");
      if (!file2) {
        Serial.println("Error al abrir el archivo soil2.txt");
      } else {
        // Escribir el valor de soil2 en el archivo y agregar una nueva línea
        for (int i = 0 ; i< 7 ; i++){
          file2.print(Measure[i]);
          file2.print(";");
        }
        file2.println();
        file2.close();
      }
    }
    
    previousMillis = currentMillis;
  }
  
}


void processSerialCommands() {
  while (Serial.available()) {
    String command = Serial.readStringUntil('\n');
    command.trim();

    if (command == "soil1") {
      // Leer y mostrar los datos del archivo soil1.txt
      Serial.println("Contenido de soil1.txt:");
      File file1 = SPIFFS.open("/soil1.txt", "r");
      if (!file1) {
        Serial.println("Error al abrir el archivo soil1.txt");
      } else {
        while (file1.available()) {
          String line = file1.readStringUntil('\n');
          Serial.println(line);
        }
        Serial.println("1122");
        file1.close();
      }
    } else if (command == "soil2") {
      // Leer y mostrar los datos del archivo soil2.txt
      Serial.println("Contenido de soil2.txt:");
      File file2 = SPIFFS.open("/soil2.txt", "r");
      if (!file2) {
        Serial.println("Error al abrir el archivo soil2.txt");
      } else {
        while (file2.available()) {
          String line = file2.readStringUntil('\n');
          Serial.println(line);
        }
        Serial.println("1122");
        file2.close();
      }
    } else if (command == "clear") {
      // Borrar contenido de los archivos soil1.txt y soil2.txt
      File file1 = SPIFFS.open("/soil1.txt", "w");
      File file2 = SPIFFS.open("/soil2.txt", "w");
      file1.close();
      file2.close();
      Serial.println("Archivos soil1.txt y soil2.txt borrados.");
    }
  }
}
