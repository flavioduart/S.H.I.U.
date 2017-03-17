#include <EEPROM.h>
#include <LiquidCrystal.h>
#define LIMITE         6          // Define o limite de erro do sinal do potenciometro
#define DEBUG          1          // Ativar(1) ou desativar(0) a comunicação com o serial.
#define ZERAR          1          // (1) zero o EEPROM (0) mantem o EEPROM com leituras anteriores
#define DELAY          500        // Define o tempo para o delay de debug em milissegundos
#define NUM_SENSORES   1          // Numero de sensores usados
#define NUM_INTERACOES 700        // Numero de interções no filtro
#define OVERFLOW       4000000000 // Over flow para o unsigned long
#define OCIO           5          // Tempo minimo entre uma ativação e outra
#define LUZ            6          // Sinalizador luminoso ligado à porta digital do arduino 
#define SOM            7          // Sirene ligada à porta digital do arduino
#define COOLER         10         // Define a porta do cooler
#define TEMPO_COOLER   3          // Tempo que o cooler permanecerá ligado 
#define NIVEL_AVISO    180        // Determina nível de ruído/pulsos para ativar o sinalizador luminoso.
#define NIVEL_LIMITE   180        // Determina nível de ruído/pulsos para ativar a sirene.
#define TEMPO_LUZ      3          // Define o tempo de duração em que o sinalizador permanecerá ativo.
#define TEMPO_SOM      3          // Define o tempo de duração em que a sirene permanecerá ativo.
#define REP_SOM        2          // Quantidade de vezes que a sirene irá disparar 
#define REP_LUZ        2          // Quantidade de vezes que o sinalizador luminoso irá disparar
#define ON             1
#define OFF            0
#define TOLERANCIA     1          // EM SEGUNDOS PELO AMOR DE DEUS!

#define slot_Acionamentos 0
int slot_limite1 = 1;
int slot_limite2 = 2;
int slot_limite3 = 3;
int slot_limite4 = 4;

int           valor = 0;
int           sensores[NUM_SENSORES] = {A2/*, A3, A5, A6*/}; // Sensores ligados às portas analógicas
int           verificadores[NUM_SENSORES] = {A0/*, A4, A1, A0*/};  // Resposansaveis por gravar saida do potenciometro
int           limite_POT[NUM_SENSORES] = {EEPROM.get(slot_limite1, valor)/*, 554, 554, 554*/};  // Variável responsável por definir o limiar do potenciometro medido analogicamente em relação à sensibilidade do sensor
bool          flag_calibracao[NUM_SENSORES];
int           nivel                  = 0;    // Variável responsável pelo nível de ruído
int           endereco               = 0;    // Endereço de memória que vai armazenar quantidade de vezes que a sirene acionou
int           Q_Acionamento          = 0;    // Variável responsável por armazenar quantidade de vezes que a sirene acionou
unsigned long t0                     = 0;    // Variável responsável pelo tempo inicial
unsigned long tc                     = 0;    // Variável responsável pelo tempo calculado
int           deveAlertar            = 1;    // Variável atua como um binário (true/false)
int botoes[] = {8, 9, 6, 7};
bool estadoBotao;
bool estadoBotao2;
void(* reset) (void) = 0;
bool verificador;
///LiquidCrystal lcd(<pino RS>, <pino enable>, <pino D4>, <pino D5>, <pino D6>, <pino D7>)
LiquidCrystal lcd(12, 11, 5, 4, 3, 2); //

//Função que define os componentes sinalizador,sirene e sensores como entrada ou saída

void setup()
{
  Serial.begin(9600);
  pinMode(LUZ,  OUTPUT); //@param[OUT]
  pinMode(SOM, OUTPUT); //@param[OUT]

  for (int i = 0; i < NUM_SENSORES; i++)
  {
    pinMode(sensores[i], INPUT); //@param[IN]
  }
  if (ZERAR)
    EEPROM.write(slot_Acionamentos, 0);
  else
    EEPROM.write(slot_Acionamentos, EEPROM.read(slot_Acionamentos));

  /*for(int i = 0; i < NUM_SENSORES; i++)
    flag_calibracao[i] = false;*/
  lcd.begin(16, 2);
  for (int i = 0; i < 4; i++)
    pinMode(botoes[i], INPUT);
}

void loop()
{
  menu();
  nivel = ouvirNivel();
  if (nivel < NIVEL_AVISO)
  {
    if (t0 < OVERFLOW)
    {
      t0 = (millis() / 1000); // Transforma para segundo para armazenar mais valores na variável
    }
    else
      reset();
  }

  deveAlertar = ((millis() / 1000) - t0) > TOLERANCIA; // Se a condicao for aceita deveAlertar = 1(true) se a condicao for falsa deveAlertar = 0(false)

  if (deveAlertar && (nivel > NIVEL_AVISO && nivel < NIVEL_LIMITE)) //se deveAlertar for verdadeiro e nivel < NIVEL_LIMITE aviso luminoso
  {
    luz();
    if (DEBUG)
      Serial.println("Em Ocio");
    else
      delay(OCIO * 1000);
  }
  else if (deveAlertar && nivel >= NIVEL_LIMITE) //se deveAlertar for verdadeiro e nivel >= NIVEL_LIMITE aviso luminoso e sonoro
  {
    luz();
    som();
    if (DEBUG)
      Serial.println("Em Ocio");
    else
      delay(OCIO * 1000);
    Q_Acionamento++;
    EEPROM.write(slot_Acionamentos, Q_Acionamento);
  }
  if (DEBUG)
  {
    Serial.print("Numero de vezes que acionou: ");
    Serial.println(EEPROM.read(slot_Acionamentos));
    delay(DELAY);
  }
}

void menu()
{
  unsigned long tempo;

  lcd.clear();
  lcd.cursor();
  lcd.blink();
  lcd.setCursor(0, 0);
  lcd.print("S.H.I.U.");
  delay(3000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Qur mudar limiar");
  lcd.setCursor(0, 1);
  lcd.print("ou sensibildade?");
  delay(4000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("1- Limiar");
  lcd.setCursor(0, 1);
  lcd.print("2- Sensibilidade");
  delay(1000);
  tempo = millis();
  do {
    estadoBotao = digitalRead(botoes[0]);
    estadoBotao2 = digitalRead(botoes[1]);
    if (estadoBotao == HIGH) {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Limiar");
      delay(2000);
      alterarlimiares();
    }
    else if (estadoBotao2 == HIGH)
    {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Sensibilidade");
      delay(2000);
      ajusteSensibilidade();
    }
  } while (estadoBotao != HIGH || estadoBotao2 != HIGH);
}

void alterarlimiares()
{
  int botao=0;
  bool botaoParar;
  int escolherBotao;
  int bin = 0;
  unsigned int alteradorDePotenciometro = 0;
  unsigned long tempo;
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Alterar limiar");
  lcd.setCursor(0, 1);
  lcd.print("c/ potenciometro");
  delay(3000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Para parar aperte");
  lcd.setCursor(0, 1);
  lcd.print("o botao 1");
  delay(3000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Escolha o sensor: ");
  lcd.setCursor(0, 1);
  lcd.print("|1|2|3|4|");

  do {
    for (int i = 0; i < NUM_SENSORES; i++) {
      estadoBotao = digitalRead(botoes[i]);
      if (estadoBotao == HIGH) {
        botao = i;
        break;
      }
    }
  } while (estadoBotao != HIGH);

  if (botao == 0) {
    delay(500);
    do {
      alteradorDePotenciometro = map(analogRead(verificadores[0]), 0, 1023, 1023, 0);
      lcd.setCursor(0, 0);
      lcd.print("New value ");
      lcd.print(alteradorDePotenciometro);
      lcd.print("   ");
      lcd.setCursor(0, 1);
      lcd.print("Old value ");
      lcd.print(limite_POT[0]);
      botaoParar = digitalRead(botoes[0]);
    } while (botaoParar != HIGH);
    EEPROM.put(slot_limite1, alteradorDePotenciometro);
    Serial.println(EEPROM.get(slot_limite1, valor));
  }
  if (botao == 1) {
    delay(500);
    do {
      alteradorDePotenciometro = map(analogRead(verificadores[1]), 0, 1023, 1023, 0);
      lcd.setCursor(0, 0);
      lcd.print("New value ");
      lcd.print(alteradorDePotenciometro);
      lcd.print("   ");
      lcd.setCursor(0, 1);
      lcd.print("Old value ");
      lcd.print(limite_POT[1]);
      botaoParar = digitalRead(botoes[0]);
    } while (botaoParar != HIGH);
    EEPROM.put(slot_limite2, alteradorDePotenciometro);
    Serial.println(EEPROM.get(slot_limite2, valor));
  }
  if (botao == 2) {
    delay(500);
    do {
      alteradorDePotenciometro = map(analogRead(verificadores[2]), 0, 1023, 1023, 0);
      lcd.setCursor(0, 0);
      lcd.print("New value ");
      lcd.print(alteradorDePotenciometro);
      lcd.print("   ");
      lcd.setCursor(0, 1);
      lcd.print("Old value ");
      lcd.print(limite_POT[2]);
      botaoParar = digitalRead(botoes[0]);
    } while (botaoParar != HIGH);
    EEPROM.put(slot_limite3, alteradorDePotenciometro);
    Serial.println(EEPROM.get(slot_limite3, valor));
  }
  if (botao == 3) {
    delay(500);
    do {
      alteradorDePotenciometro = map(analogRead(verificadores[3]), 0, 1023, 1023, 0);
      lcd.setCursor(0, 0);
      lcd.print("New value ");
      lcd.print(alteradorDePotenciometro);
      lcd.print("   ");
      lcd.setCursor(0, 1);
      lcd.print("Old value ");
      lcd.print(limite_POT[3]);
      botaoParar = digitalRead(botoes[0]);
    } while (botaoParar != HIGH);
    EEPROM.put(slot_limite4, alteradorDePotenciometro);
    Serial.println(EEPROM.get(slot_limite4, valor));
  }

  do {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("1 - Limiar");
    lcd.setCursor(0, 2);
    lcd.print("2 - Sair");
    delay(500);
    if (digitalRead(botoes[0]) == HIGH)
     alterarlimiares();
    else if (digitalRead(botoes[1]) == HIGH)
     menu(); 
  } while (digitalRead(botoes[0]) != HIGH || digitalRead(botoes[1]) != HIGH);
}

int ouvirNivel()
{
  int value = 0;
  int leitura_sensor = 0;
  int leitura_verificador = 0;
  for (int i = 0; i < NUM_SENSORES; i++)
  {
    if (flag_calibracao[i]) {
      leitura_sensor = read_sensor(sensores[i]);
      //leitura_verificador = read_sensor(verificadores[i]);
      if (DEBUG)
      {
        imprime_valor(i, leitura_sensor);
        //imprime_verificador(i, leitura_verificador);
      }
      if (leitura_sensor > value)
        value = leitura_sensor;
    }
  }
  return value;
}

int read_sensor(int port)
{
  unsigned long value = 0;
  for (int x = 0; x < NUM_INTERACOES; x++) //Antes tinha times, uma funcao que passava como argumento. Desnecessario muito mais viavel usar um define N_INTERACOES
  {
    value += map(analogRead(port), 0, 1023, 1023, 0); //Inverte os valores começando em 1023 e terminando em 0
    //Especificamente esse sensor utiliza 1023 como silencio e 0 como barulho máximo.
  }
  value /= NUM_INTERACOES;
  return value;
}

void imprime_valor(int x, int leitura)
{
  Serial.print("Sensor ");
  Serial.print(x + 1);
  Serial.print(": ");
  Serial.println(leitura);
}
void luz()
{
  for (int n = 0 ; n < REP_LUZ; n++)
  {
    if (DEBUG)
      Serial.println("Disparado Luz");
    else
    {
      digitalWrite(LUZ, HIGH);
      delay(TEMPO_LUZ * 1000);
      digitalWrite(LUZ, LOW);
    }
    if (DEBUG)
      Serial.println("Desligando Luz");
  }
}

void som()
{
  for (int n = 0; n < REP_SOM ; n++ )
  {
    if (DEBUG)
      Serial.println("Disparado Som");
    else
    {
      digitalWrite(SOM, HIGH);
      delay(TEMPO_SOM * 1000);
      digitalWrite(SOM, LOW);
    }
    if (DEBUG)
      Serial.println("Desligando Som");
  }
}

void imprime_verificador(int y, int leitura)
{
  Serial.print("Verficador ");
  Serial.print(y + 1);
  Serial.print(" : ");
  Serial.println(leitura);
  Serial.println();
}


//--------------------- FUNÇÃO CRIADA PARA AUXILIAR A AJUSTAR A SENSIBILIDADE DO POTENCIOMETRO --------------------
// OBS.: PRECISA TER VERIFICADO O LIMIAR DO POTENCIOMETRO PARA CADA SENSOR E INSERIDO NO CODIGO

// COM BUGS


void ajusteSensibilidade() {                     //A função recebe uma porta analogica (de um potenciometro) como parametro, para realizar a calibração do sensor
  bool ajuste = false;                           //Flag para a calibração. Regulada - True; Desregulada - False;
  int leitura = 0;                                   //Variavel de leitura analogica da porta
  int botao = 0;
  int value = 0;

  String str = "Sensores";
  String sensor = "";
  for (int i = 0; i < NUM_SENSORES; i++) {
    leitura = read_sensor(verificadores[i]);
    value = leitura - limite_POT[i];
    if (value > LIMITE || value < -LIMITE) {
      flag_calibracao[i] = false;
    } else {
      flag_calibracao[i] = true;
    }
    if (flag_calibracao[i] == false)
      sensor += " " + String(i + 1) + ",";
  }

  if (sensor == "") {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print(str + " Calibrados");
    delay(2000);
    lcd.clear();
    menu();
  } else {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print(str + sensor);
    lcd.setCursor(0, 1);
    lcd.print("Descalibrados");
    delay(4000);
  }


  if (flag_calibracao[0] == false) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Digite o numero");
    lcd.setCursor(0, 1);
    lcd.print("do botao");

    do {
      for (int i = 0; i < NUM_SENSORES; i++) {
        estadoBotao = digitalRead(botoes[i]);
        if (estadoBotao == HIGH) {
          botao = i;
          break;
        }
        break;
      }
    } while (estadoBotao != HIGH);

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Apos termino");
    lcd.setCursor(0, 1);
    lcd.print("aperte o msm bot");

    delay(2000);
    while (!flag_calibracao[botao]) {       //Laço para calibrar todos os sensores listados;
      leitura = read_sensor(verificadores[botao]);
      value = leitura - limite_POT[botao];         //Variavel de analise da precisão da calibração
      if (value > LIMITE || value < -LIMITE) {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("limite: ");
        lcd.print(limite_POT[botao]);
        lcd.setCursor(0, 1);
        lcd.print("pot: ");
        lcd.print(value);
      } else
        lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("limite: ");
      lcd.print(limite_POT[botao]);
      lcd.setCursor(0, 1);
      lcd.print("pot: ");
      lcd.print(value);
      lcd.print(" calibrado");
      estadoBotao = digitalRead(botoes[botao]);
      if (estadoBotao == HIGH)
        flag_calibracao[botao] = true;
    }
  }
}
