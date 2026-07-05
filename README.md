#include <SoftwareSerial.h>

SoftwareSerial gsm(7, 8);      // RX, TX

#define GAS_SENSOR A0
#define BUZZER 3
#define LED 4
#define RELAY 5

int gasValue = 0;
int threshold = 400;
int smsSent = 0;

void setup()
{
    Serial.begin(9600);
    gsm.begin(9600);

    pinMode(GAS_SENSOR, INPUT);
    pinMode(BUZZER, OUTPUT);
    pinMode(LED, OUTPUT);
    pinMode(RELAY, OUTPUT);

    digitalWrite(BUZZER, LOW);
    digitalWrite(LED, LOW);

    // Relay ON (Power Supply Available)
    digitalWrite(RELAY, HIGH);
}

void loop()
{
    gasValue = analogRead(GAS_SENSOR);

    Serial.print("Gas Value: ");
    Serial.println(gasValue);

    if(gasValue > threshold)
    {
        digitalWrite(BUZZER, HIGH);
        digitalWrite(LED, HIGH);

        // Cut OFF Power Supply
        digitalWrite(RELAY, LOW);

        if(smsSent == 0)
        {
            SendSMS();
            smsSent = 1;
        }
    }
    else
    {
        digitalWrite(BUZZER, LOW);
        digitalWrite(LED, LOW);

        // Restore Power
        digitalWrite(RELAY, HIGH);

        smsSent = 0;
    }

    delay(500);
}

void SendSMS()
{
    gsm.println("AT");
    delay(1000);

    gsm.println("AT+CMGF=1");
    delay(1000);

    gsm.println("AT+CMGS=\"+91XXXXXXXXXX\"");
    delay(1000);

    gsm.print("ALERT! GAS LEAKAGE DETECTED.");
    gsm.print(" POWER SUPPLY TURNED OFF.");

    gsm.write(26);

    delay(5000);
}
