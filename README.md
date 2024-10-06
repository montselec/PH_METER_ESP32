Hola bienvenidos PLCTermineitors! Os dejo el tutorial el cual se describe como calibrar con la ayuda del gran greenponik el cual he modificado unos valores en su codigo para poder calibrar mi sensor de PH, os dejo conexionado,
programa de Arduino, y lo proximo que realizaré es enlace al sistema domótico home assistant.

##CALIBRACION##
Fisicamente hay que cortocircuitar el conector BNC conectando un hilo en el orificio central y otro a la carcasa del conector, ahora con un multimetro medimos en paralelo entre , PO y GND de la placa del ESP32, debe de marcar 2,5V que esto equivale a un PH de 7 neutro.
Adjuntar fotografia:


Si está el voltaje muy desviado puedes regular con el potenciometro que trae mas cercano al conector BNC ese voltaje.

Otra opción es hacer la calibracion mediante software que es la que me ha tocado hacer a mi ya que mi valor en voltaje es muy alto, desconozco la razón no se si es por el sensor que se ha llevado un tiempo sin utilizar.
Vamos a calibrar mediante codigo:
![imagen](https://github.com/user-attachments/assets/797cafac-afa5-4a4f-aa3f-74764dc6f807)

Aqui como veis estos valores han sido todo modificados, para ello solo tenemos que fijarnos en el Monitor Serie que valor nos dá con una solucion de PH 7 como por ejemplo el agua del grifo
![imagen](https://github.com/user-attachments/assets/e76e7c97-9eb5-4d30-b123-7d3e08ba9f55)
Sabiendo que voltaje obtenemos con agua del grifo ahora nos toca modificar el codigo siguiente para que acepte la calibración:
En mi caso tambien use una solucion de PH 4 para realizar el "slope" (pendiente de linealidad del sensor)  correcto:
Una vez que tengamos los valores del voltaje con cada solucion lo insertamos , y ahora si que nos ponemos a calibrarlo, en la barra de comandos del monitor serie escribimos:
ENTERPH saldrá un mensaje "Enter PH Calibration Mode" "Please put the probe into the 4.0 or 7.0 standard buffer solution<<<" y aquí es donde está el "truco", Debemos de hacerle creer mediante software que ese valor es el que corresponde al PH 7 por ejemplo entonces en la siguiente parte del codigo modificamos:
```ruby
 if (phCalibrationFinish)
            {
                if ((this->_voltage > PH_8_VOLTAGE) && (this->_voltage < PH_5_VOLTAGE))
                {
                    EEPROM.writeFloat(PHVALUEADDR, this->_neutralVoltage);
                    EEPROM.commit();
                }
                else if ((this->_voltage > PH_5_VOLTAGE) && (this->_voltage < PH_3_VOLTAGE))
                {
                    EEPROM.writeFloat(PHVALUEADDR + sizeof(float), this->_acidVoltage);
                    EEPROM.commit();
                }
                Serial.print(F(">>>Calibration Successful"));
```



####CREDITOS#####
Post original y al que muestro un gran agradecimiento:
https://github.com/greenponik/DFRobot_ESP_PH_BY_GREENPONIK

Y a su vez a su autor oficial 
Written by Mickael Lehoux from @greenponik
Based on written by Jiawei Zhang(Welcome to our website)
