# Zdalnie sterowany pojazd przez smatfona
Projekt zakłada zbudowanie pojazdu (w tym przypadku samochodziku), który przy użyciu smartfona (na androidzie) będzie można sterować pojazdem przez bluetooth.

### Użyte elementy:
  - Arduino Uno
  - L298N (moduł do sterowania silnikami)
  - HC-05 (moduł bluetooth)
  - 2x silniki (L298N może maksymalnie pracować na 48V i obsługuje silniki do max 2A)
  - bateria 9V (ale można użyć innego źródła zasilania do czego zachęcam aby zwiększyc wydajność silników)
  - trochę przewodów
  - szkielet jakiegoś pojazdu (może to być jakiś stary samochodzik)
  - trochę cierpliwości 😜

### Zastosowany schemat:

![schemat samochody bt v1_bb](https://user-images.githubusercontent.com/93213017/153260119-1e250e31-ca7c-4402-b123-5abc05e6c8df.jpg)

### Aplikacja na telefon

![6ain59(2)](https://user-images.githubusercontent.com/93213017/153293470-ce78e69a-5e88-4fc2-affb-07d0d1c63824.png)

https://play.google.com/store/apps/details?id=braulio.calle.bluetoothRCcontroller

Jak połączyć się z modułem bluetooth?
  1. Pobieramy aplikacje
  2. Na smartfonie włączamy bluetooth
  3. W wyszukiwaniu urządzeń na bluetooth szukamy urządzenia HC-05, hasło to: 1234
  4. Włączamy aplikacje
  5. Naciskamy na zębadkę
  6. Po otworzeniu się "Option Menu" naciskamy "Connect to car" i wybieramy "HC-05"
  7. Powinien pokazać się komunikat o sukcesie połączenia z urządzeniem

### Kod programu:
*Został napisany dla Arduino, kod został dostosowany do pokazanej powyżej aplikacji Link do oprogramowania arduino* 

https://www.arduino.cc/en/software

```
// TX RX bibliotego software dla bluetooth
#include <SoftwareSerial.h> 

//inicjalizacja pinow dla modulu bluetooth
int bluetoothTx = 2; // bluetooth tx - 2 pin
int bluetoothRx = 3; // bluetooth rx - 3 pin
SoftwareSerial bluetooth(bluetoothTx, bluetoothRx);

//zdefiniowanie pinow arduno - nadanie pinom nazw
//piny dla glownego silnika  
int Enable1 = 5;
int Motor1_Pin1 = 6;  
int Motor1_Pin2 = 7;  

//piny dla silnika sterujacego   
int Motor2_Pin1 = 8; 
int Motor2_Pin2 = 9;
int Enable2 = 10; 


char command ; //zmienna w celu przechowywania danych
int velocity = 0; //zmienna velocity ktora bedzie odpowiadac za predkosc obrotow silnikow

//ustawienie czestotliwosci modulu bluetooth
void setup(){       
  Serial.begin(9600);  
  bluetooth.begin(9600);

  //ustawienie wyjsc z arduino (podobnie jak przy zdefiniowaniu pinow)
  pinMode(Motor1_Pin1, OUTPUT);  
  pinMode(Motor1_Pin2, OUTPUT);
  pinMode(Enable1, OUTPUT);
  
  pinMode(Motor2_Pin1, OUTPUT);  
  pinMode(Motor2_Pin2, OUTPUT);
  pinMode(Enable2, OUTPUT); 
}

void loop(){
  if(bluetooth.available() > 0){  //sprawdzenie, czy dostępne sa jakies dane
    command = bluetooth.read();   //przechowywanie danych w zmiennej „polecenie”
    Serial.println(command);      //uzycie zmiennej
    {
    case 'F':  //do przodu
      digitalWrite(Motor2_Pin2, LOW);
      digitalWrite(Motor2_Pin1, HIGH);
      digitalWrite(Motor1_Pin1, LOW);
      digitalWrite(Motor1_Pin2, LOW);
      break;
    case 'B':  //do tylu
      digitalWrite(Motor2_Pin1, LOW);
      digitalWrite(Motor2_Pin2, HIGH);
      digitalWrite(Motor1_Pin1, LOW);
      digitalWrite(Motor1_Pin2, LOW);
      break;
    case 'L':  //w lewo
      digitalWrite(Motor1_Pin1, LOW);
      digitalWrite(Motor1_Pin2, HIGH);
      digitalWrite(Motor2_Pin1, LOW);
      digitalWrite(Motor2_Pin2, LOW);
      break;
    case 'R':   //w prawo
      digitalWrite(Motor1_Pin2, LOW);
      digitalWrite(Motor1_Pin1, HIGH);  
      digitalWrite(Motor2_Pin1, LOW);
      digitalWrite(Motor2_Pin2, LOW);
      break;
    case 'S':   //stop
      digitalWrite(Motor2_Pin2, LOW);
      digitalWrite(Motor2_Pin1, LOW);
      digitalWrite(Motor1_Pin2, LOW);
      digitalWrite(Motor1_Pin1, LOW);
      break; 
    case 'I':  //do przodu w prawo
      digitalWrite(Motor2_Pin2, LOW);
      digitalWrite(Motor2_Pin1, HIGH);
      digitalWrite(Motor1_Pin2, LOW);
      digitalWrite(Motor1_Pin1, HIGH);
      break; 
    case 'J':  //do tylu w prawo
      digitalWrite(Motor1_Pin2, LOW);
      digitalWrite(Motor1_Pin1, HIGH);
      digitalWrite(Motor2_Pin1, LOW);
      digitalWrite(Motor2_Pin2, HIGH);
      break;        
    case 'G':  //do przodu w lewo
      digitalWrite(Motor2_Pin2, LOW);
      digitalWrite(Motor2_Pin1, HIGH);
      digitalWrite(Motor1_Pin1, LOW);
      digitalWrite(Motor1_Pin2, HIGH);
      break; 
    case 'H':  //do tylu w lewo
      digitalWrite(Motor2_Pin1, LOW);
      digitalWrite(Motor2_Pin2, HIGH);
      digitalWrite(Motor1_Pin1, LOW);
      digitalWrite(Motor1_Pin2, HIGH);
      break;

    //kontrola predkosci
    default:  //uzycie zmiennej velocity
      if(command=='q'){
        velocity = 255;  //maksymalna wartosc zmiennej
        analogWrite(Enable1, velocity);
        analogWrite(Enable2, velocity);
      }
      else{ 
        //znaki '0' - '9' maja wartosc odpowiednio 48-57
        if((command >= 48) && (command <= 57)){ 
          //odjecie 48 zmienia zakres z 48-57 na 0-9
          //mnozenie przez 25 zmienia zakres od 0-9 do 0-225
          velocity = (command - 48)*25;       
          analogWrite(Enable1, velocity);
          analogWrite(Enable2, velocity);
        }
      }
    }
  }
}
```

### Finalny efekt:

### Przemyślenia:
Kod można jak i sam samochodzik wzbogacić o klakson oraz oświetlenie.
Zasilanie pojazdu bym rozdzielił na 9V i 12V-48V dla modułu L298N (zwiększy to moc dla silników)
