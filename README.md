Hola bienvenidos PLCTermineitors! Os dejo el tutorial el cual se describe como conectar, programar y  calibrar con la ayuda del gran greenponik el cual he modificado unos valores en su codigo para poder calibrar mi sensor de PH, os dejo conexionado,
programa de Arduino, y lo proximo que realizaré es enlace al sistema domótico home assistant.

# ESQUEMA ELECTRICO #
Como veis es bastante facil realizar el conexionado electrico para esta aplicacion:
Segun el fabricante del sensor de PH dejo una descripcion de cada punto:
TO – SALIDA DE TEMPERATURA
DO – LIMITE DE LA SALIDA ALARMA A > 3.3V
PO – SALIDA ANALOGICA DE PH
GND – MASA PARA SEÑAL DE PH 
GND – MASA DE LA PLACA
VCC – 5V DC
POT1 – CALIBRACION DE LA SALIDA ANALOGICA DE PH (EL MAS CERCANO AL CONECTOR BNC)
POT2 – AJUSTE DE ALARMA SALIDA DO DIGITAL.


![imagen](https://github.com/user-attachments/assets/f158ccf5-a537-42e4-8772-57b30aa3b0da)










![esquema de conexion PH](https://github.com/user-attachments/assets/1c78873e-8d70-40aa-bd6f-3d10f0d0015f)







#CALIBRACION#

Fisicamente hay que cortocircuitar el conector BNC conectando un hilo en el orificio central y otro a la carcasa del conector, ahora con un multimetro medimos en paralelo entre , PO y GND de la placa del ESP32, debe de marcar 2,5V que esto equivale a un PH de 7 neutro.
Adjuntar fotografia:
![how-to-calibrate-the-ph-sensor-1280x694](https://github.com/user-attachments/assets/3d3e82d8-a3f2-4a93-a8bc-66b94df1f9fe)
Gracias a elecroclinic por su post y explicaciones.
https://www.electroniclinic.com


Si está el voltaje muy desviado puedes regular con el potenciometro que trae mas cercano al conector BNC ese voltaje.

Otra opción es hacer la calibracion mediante software que es la que me ha tocado hacer a mi ya que mi valor en voltaje es muy alto, desconozco la razón no se si es por el sensor que se ha llevado un tiempo sin utilizar.
Vamos a calibrar mediante codigo:
1.![imagen](https://github.com/user-attachments/assets/797cafac-afa5-4a4f-aa3f-74764dc6f807)

Aqui como veis estos valores han sido todo modificados, para ello solo tenemos que fijarnos en el Monitor Serie que valor nos dá con una solucion de PH 7 como por ejemplo el agua del grifo
![imagen](https://github.com/user-attachments/assets/e76e7c97-9eb5-4d30-b123-7d3e08ba9f55).

Sabiendo qué voltaje obtenemos con agua del grifo ahora nos toca modificar el codigo siguiente para que acepte la calibración:
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

Las variables PH_8_VOLTAGE y PH_5_Voltage hacen referencia al voltaje modificado en el apartado anterior  ( 1 ), entonces indica si el voltaje actual es mayor al valor que hayamos puesto en la variable PH_8_VOLTAGE y ademas inferior al valor que hemos puesto en PH_5_VOLTAGE  obetenemos una calibración valida para PH 7  :D .

Por ultimo ejecutamos CALPH nos indicara que ha sido satisfactorio, y volvemos a insertar el comando EXITPH para guardar el valor de calibración.

Para la calibración de PH 4 es muy similar tenemos un entonces si lo anterior no es valido, y el voltaje que obtenemos es mayor al valor de tension que hemos metido en la variable PH_5_VOLTAGE y ademas es inferior al valor insertado en la variable  PH_3_VOLTAGE ese valor se convierte en PH 4, Los pasos desde un principio seria:
1.ENTERPH en monitor serie ( Entramos en modo calibración)
2.CALPH ( ya con la sonda de ph insertada en el patron 4 de la solucion).
3.EXITPH en el monitor serie ( para guardar la calibración)  
Ya tendriamos un slope de 4 a 7 de PH bien calibrado :D.



```ruby
                else if ((this->_voltage > PH_5_VOLTAGE) && (this->_voltage < PH_3_VOLTAGE))
                {
                    EEPROM.writeFloat(PHVALUEADDR + sizeof(float), this->_acidVoltage);
                    EEPROM.commit();
                }
                Serial.print(F(">>>Calibration Successful"));


```


#¿Como conectarlo a home assistant?#

Para conectarlo a HomeAssistant, tenemos que tener las siguientes librerias instaladas en Arduino, en el código que dejo tenéis perfectamente todas las que son necesarias:





#CREDITOS#

Post original y al que muestro un gran agradecimiento:
https://github.com/greenponik/DFRobot_ESP_PH_BY_GREENPONIK
Creditos a electroclinic por su enorme contribucion y explicacion de paso a paso de su sensor de PH:
https://www.electroniclinic.com/ph-meter-arduino-ph-meter-calibration-diymore-ph-sensor-arduino-code/
Y a su vez a su autor oficial 
Written by Mickael Lehoux from @greenponik
Based on written by Jiawei Zhang(Welcome to our website)
