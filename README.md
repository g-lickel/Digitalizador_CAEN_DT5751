G. Lickel - 17 fev. 2024 (Rev. 0)

# Tutorial de instalação e uso do Digitalizador da CAEN DT5751 no Ubuntu 20.04
## Disclaimer
This is an unofficial tutorial for simple installation and use, with absolutely no connection to CAEN. For official support, access the [caen.it](caen.it) website.

## Instalação
### Pré-requisitos
Antes de instalar os softwares de leitura do digitalizador, é necessário alguns pré-requisitos.
1. Verifique se os softwares `gnuplot` e `DKMS` estão instalados no seu Linux. Se não estiverem, instale-os.
2. Entre no site da CAEN, [caen.it](https://www.caen.it/become-mycaenplus-user/), e crie uma conta gratuitamente. Essa conta será necessária para baixar os programas.
3. No site, baixe a biblioteca [CAENVMELib](https://www.caen.it/products/caenvmelib-library/). Para o Linux, baixe o arquivo `.tgz`.
4. Baixe também o arquivo `.tgz` da biblioteca [CAENComm](https://www.caen.it/products/caencomm-library/).
5. Baixe o arquivo `.tgz` da biblioteca [CAENDigitizer](https://www.caen.it/products/caendigitizer-library/).
6. Por fim, baixe o driver `.tar.gz` do Digitalizador [DT5751](https://www.caen.it/download/?filter=DT5751).
Supondo que tenha baixado na pasta `~/Downloads`, faça o seguinte para instalar as bibliotecas.
```bash
cd ~
mkdir CAEN_Digitizer && cd CAEN_Digitizer
tar -xzvf ~/Downloads/CAENVMELib-v4.0.2.tgz
tar -xzvf ~/Downloads/CAENComm-v1.7.0.tgz
tar -xzvf ~/Downloads/CAENDigitizer-v2.17.3.tgz
tar -xzvf ~/Downloads/CAENUSBdrvB-v1.6.1.tar.gz
cd ~/CAEN_Digitizer/CAENVMELib-v4.0.2/lib/
sudo ./install_x64
cd ~/CAEN_Digitizer/CAENComm-v1.7.0/lib/
sudo ./install_x64
cd ~/CAEN_Digitizer/CAENDigitizer-v2.17.3/lib/
sudo ./install_x64
cd ~/CAEN_Digitizer/CAENUSBdrvB-v1.6.1/
sudo ./install.sh
```
Se os programas rodarem sem erro, as bibliotecas e drivers devem ter sido instalados com êxito.

### WaveDump e CAENSCOPE
Em nosso tutorial, usaremos dois softwares da CAEN.

7. Novamente no site da CAEN, instale o arquivo `.tar.gz` do software [WaveDump](https://www.caen.it/products/caen-wavedump/).
8. Em seguida, baixe o `.tar.gz` do [CAENSCOPE](https://www.caen.it/products/caenscope/).

Para instalar o WaveDump, continue na pasta `CAEN_Digitizer` no terminal
```bash
cd ~/CAEN_Digitizer/
tar -xzvf ~/Downloads/wavedump-3.10.6.tar.gz
cd wavedump-3.10.6/
./configure
make
sudo make install
```

Para instalar o CAENSCOPE, basta apenas
```bash
cd ~/CAEN_Digitizer/
mkdir CAENScope_v1.3_Linux-x64 && cd CAENScope_v1.3_Linux-x64
tar -xzvf ~/Downloads/CAENScope_v1.3_Linux-x64.tar.gz
```
#### Atalhos opcionais
##### Desktop
Para facilitar a abertura do Scope, você pode criar um atalho na área de trabalho. Para isso, siga na pasta `CAEN_Digitizer`
```bash
cd ~/CAEN_Digitizer/CAENScope_v1.3_Linux-x64/
wget https://www.caen.it/wp-content/uploads/2017/10/CAEN_Scope.png
cd ~/Desktop # ou ~/Área\ de\ Trabalho em português
gedit scope.desktop &
```
No documento aberto, escreva (substituindo `<username>` pelo seu usuário no Ubuntu)
```desktop
[Desktop Entry]
Name=CAENSCOPE
Exec=/home/<username>/CAEN_Digitizer/CAENScope_v1.3_Linux-x64/bin/scope-l64
Icon=/home/<username>/CAEN_Digitizer/CAENScope_v1.3_Linux-x64/CAEN_Scope.png
Terminal=false
Type=Application
```
Salve e feche o documento. De volta ao terminal, dê permissão de executável ao atalho
```bash
chmod +x ~/Desktop/scope.desktop
```
##### Criar executável
Outra forma é criar um comando `scope` para executar o aplicativo em qualquer diretório. Para isso, faça o seguinte
```bash
gedit ~/scope.sh
```
No editor de texto, escreva
```bash
#!/bin/bash
exec /home/<username>/CAEN_Digitizer/CAENScope_v1.3_Linux-x64/bin/scope-l64 "$@"
```
Salve e feche o bloco de notas. Em seguida, escreva no terminal
```bash
chmod +x ~/scope.sh
sudo mv ~/scope.sh /usr/local/bin/scope
```
Pronto. Para executar o aplicativo em segundo plano, basta escrever no terminal
```bash
scope &
```

## Uso do Digitalizador
Neste exemplo, estaremos usando o digitalizador ligado à uma "raquete" de PMT com cintilador. Portanto, os sinais medidos serão **negativos**. Para sinais positivos, adaptar o tutorial levando isso em consideração.
### Abrindo no Scope
Abra o Scope (pelo atalho criado ou pelo terminal)
```bash
cd ~
./CAEN_Digitizer/CAENScope_v1.3_Linux-x64/bin/scope
```

1. Com o digitalizador ligado e conectado ao anodo da PMT, ligue ao computador pelo cabo USB. Em seguida, na tela aberta do Scope, vá em `File > Connect... > Ok` (supondo que por padrão esteja `usb 0 0`).
2. Na nova janela aberta, veja na direita a seção `Trigger Settings` e selecione o `Trigger Mask` em `0`. Logo abaixo, selecione a opção `Auto` e dê `Start` no canto inferior direito da tela.
3. Com isso, você estará vendo a linha de *offset* do detector, apesar de não enxergar sinal nenhum visto que o *threshold* está zerado.
4. Agora, use o botão `DC Offset` na barra inferior, descendo seu valor até que essa faixa de ruído esteja mais alta possível (próxima da segunda marcação horizontal do *grid*). No meu exemplo, ficou com um valor de `8630.0`, mas varia de acordo com o detector.
5. Em seguida, mude o botão de `Threshold` na barra direita, subindo seu valor até que a setinha preta na tela indique um bom valor de *threshhold/trigger* (nesse momento, você deve estar enxergando o seu sinal esperado). No meu exemplo, esse valor ficou em `950.0`. Por fim, mude a seleção de `Auto` feita anteriormente para `Normal`.
6. Para melhorar a visualização, você pode ir na barra superior e clicar em `View > Calibrate Axes` para visualizar o eixo horizontal em nanossegundos.

Pronto, a leitura pelo Scope está configurada. Se quiser, você poderá fazer a aquisição por este aplicativo, mas vamos usar o WaveDump para isso.

### Configurando o WaveDump
Antes de tudo, anote os valores de `DC Offset` e `Threshold` do Scope e feche o programa.

No terminal
```bash
cd ~
gedit /etc/wavedump/WaveDumpConfig.txt &
```
Este é o arquivo de configuração do WaveDump. Vamos alterar os seguintes parâmetros, a partir do arquivo padrão:

1. Como se trata de uma leitura de sinal negativo, vamos mudar o parâmetro `PULSE_POLARITY` para `NEGATIVE`.
2. No final do documento, abaixo da linha `[0]`, vamos alterar os parâmetros para leitura. A partir do valor de DC Offset anotado do Scope, determinamos . O resultado deve ser um número entre 0 e 100, representando a porcentagem da baseline. Em nosso exemplo, teremos BASELINE_LEVEL         13 (dado que a conta dá 13,17%)
3. A partir do valor de `Threshold`, determinamos `TRIGGER_THRESHOLD`. Como configuramos a valor do `DC Offset` de forma que a *baseline* ficasse mais próxima possível de `1000` ADC, então $\mathtt{TRIGGER\_THRESHOLD} = 1000 - \mathtt{Threshold}$. Neste exemplo, teremos TRIGGER_THRESHOLD      50.

Por fim, o arquivo `WaveDumpConfig.txt` deve ficar
```wavedump
# ****************************************************************
# WaveDump Configuration File
# ****************************************************************

# NOTE:
# The lines between the commands @OFF and @ON will be skipped.
# This can be used to exclude parts of the file.

# ----------------------------------------------------------------
# Settings common to all channels
# ----------------------------------------------------------------
[COMMON]

# OPEN: open the digitizer
# options: USB 0 0      			Desktop/NIM digitizer through USB              
#          USB 0 BA     			VME digitizer through USB-V1718/V3718 (BA = BaseAddress of the VME board, 32 bit hex)
#          PCI 0 0 0    			Desktop/NIM/VME through CONET (optical link) 
#          PCI 0 0 BA   			VME digitizer through V2718/V3718 (BA = BaseAddress of the VME board, 32 bit hex)
#          USB_A4818 X 0 0			Desktop/NIM digitizer through USB->A4818->CONET (X is the PID (product id) of A4818)
#          USB_A4818_V2718 X 0 BA   VME digitizer through USB-A4818-V2718 (BA = BaseAddress of the VME board, 32 bit hex) (X is the PID (product id) of A4818)
#          USB_A4818_V3718 X 0 BA   VME digitizer through USB-A4818-V3718 (BA = BaseAddress of the VME board, 32 bit hex) (X is the PID (product id) of A4818)
OPEN USB 0 0
#OPEN USB_A4818 12345 0 0 
#OPEN USB_A4818_V2718 12345 0 32100000
#OPEN USB 0 32100000
#OPEN PCI 0 0 0
#OPEN PCI 0 0 32100000

# RECORD_LENGTH = number of samples in the acquisition window
RECORD_LENGTH  1024

# DECIMATION_FACTOR: ONLY FOR 740 and 724 MODELS. change the decimation factor for the acquisition.
# options: 1 2 4 8 16 32 64 128  
DECIMATION_FACTOR  1

# POST_TRIGGER: post trigger size in percent of the whole acquisition window
# options: 0 to 100
# On models 742 there is a delay of about 35nsec on signal Fast Trigger TR; the post trigger is added to
# this delay  
POST_TRIGGER  50

#PULSE_POLARITY: input signal polarity.
#options: POSITIVE, NEGATIVE
#
PULSE_POLARITY  NEGATIVE

# EXTERNAL_TRIGGER: external trigger input settings. When enabled, the ext. trg. can be either 
# propagated (ACQUISITION_AND_TRGOUT) or not (ACQUISITION_ONLY) through the TRGOUT
# options: DISABLED, ACQUISITION_ONLY, ACQUISITION_AND_TRGOUT
EXTERNAL_TRIGGER   DISABLED	

# FPIO_LEVEL: type of the front panel I/O LEMO connectors 
# options: NIM, TTL
FPIO_LEVEL  NIM

# OUTPUT_FILE_FORMAT: output file can be either ASCII (column of decimal numbers) or binary 
# (2 bytes per sample, except for Mod 721 and Mod 731 that is 1 byte per sample)
# options: BINARY, ASCII
OUTPUT_FILE_FORMAT  ASCII

# OUTPUT_FILE_HEADER: if enabled, the header is included in the output file data
# options: YES, NO
OUTPUT_FILE_HEADER  YES

# TEST_PATTERN: if enabled, data from ADC are replaced by test pattern (triangular wave)
# options: YES, NO
TEST_PATTERN   NO

# WRITE_REGISTER: generic write register access. This command allows the user to have a direct write access
# to the registers of the board. NOTE: all the direct write accesses are executed AFTER the other settings,
# thus it might happen that the direct write overwrites a specific setting.
# To avoid this use the right "MASK".
# Syntax: WRITE_REGISTER ADDRESS DATA MASK, where ADDRESS is the address offset of the register (16 bit hex), DATA
# is the value being written (32 bit hex) and MASK is the bitmask to be used for DATA masking.
# Example: Set only bit [8] of register 1080 to 1, leaving the other bits to their previous value
# WRITE_REGISTER 1080 0100 0100
# Example: Set only bit [8] of register 1080 to 0, leaving the other bits to their previous value
# WRITE_REGISTER 1080 0000 0100
# Example: Set register 1080 to the value of 0x45:
# WRITE_REGISTER 1080 45 FFFFFFFF

# ----------------------------------------------------------------
# Individual Settings 
# ----------------------------------------------------------------
# The following setting are usually applied on channel by channel
# basis; however, you can put them also here in the [COMMON] section in
# order to apply them to all the channels.
# ----------------------------------------------------------------

# ENABLE_INPUT: enable/disable one channel
# options: YES, NO
ENABLE_INPUT          NO

#BASELINE_LEVEL: baseline position in percent of the Full Scale. 
# POSITIVE PULSE POLARITY (Full Scale = from 0 to + Vpp)
# 0: analog input dynamic range = from 0 to +Vpp 
# 50: analog input dynamic range = from +Vpp/2 to +Vpp 
# 100: analog input dynamic range = null (usually not used)*
# NEGATIVE PULSE POLARITY (Full Scale = from -Vpp to 0) 
# 0: analog input dynamic range = from -Vpp to 0 
# 50: analog input dynamic range = from -Vpp/2 to 0 
# 100: analog input dynamic range = null (usually not used)*
#
# options: 0 to 100
# NOTE: reasonable values should keep a margin of 10%, otherwise the
# actual baseline level may differ from the specified one.
BASELINE_LEVEL  50

# TRIGGER_THRESHOLD: threshold for the channel auto trigger (ADC counts)
# options 0 to 2^N-1 (N=Number of bit of the ADC)
# *The threshold is relative to the baseline:
# 	POSITIVE PULSE POLARITY: threshold = baseline + TRIGGER_THRESHOLD
# 	NEGATIVE PULSE POLARITY: threshold = baseline - TRIGGER_THRESHOLD
#
TRIGGER_THRESHOLD      100

# CHANNEL_TRIGGER: channel auto trigger settings. When enabled, the ch. auto trg. can be either 
# propagated (ACQUISITION_AND_TRGOUT) or not (ACQUISITION_ONLY) through the TRGOUT
# options: DISABLED, ACQUISITION_ONLY, ACQUISITION_AND_TRGOUT, TRGOUT_ONLY
# NOTE: since in x730 boards even and odd channels are paired, their 'CHANNEL_TRIGGER' value
# will be equal to the OR combination of the pair, unless one of the two channels of
# the pair is set to 'DISABLED'. If so, the other one behaves as usual.
CHANNEL_TRIGGER        ACQUISITION_ONLY

#In the following, you can see the use of some individual settings to:
#	-enable channel [0]
#	-position the baseline to 10% of the full scale, to use the input dynamic range in a better way
#	-set the trigger threshold of channel [0] to 50 LSB (relative to the baseline position)   
# 

[0]
ENABLE_INPUT           YES
BASELINE_LEVEL         13
TRIGGER_THRESHOLD      50

[1]


[2]


[3]
```

Salvando o arquivo, você pode executar o to WaveDump. Por exemplo
```bash
mkdir ~/Medidas\ teste && cd ~/Medidas\ teste
wavedump 
```
Tendo conectado com êxito, inicie as medidas clicando `s`. Ao clicar `Shift+P` você pode visualizar as medições serem plotadas em tempo real.

Para salvar vários eventos, clique `Shift+W` e depois `Shift+W` para parar a gravação. Os eventos registrados estarão em um arquivo de texto na pasta `/Medidas teste/` que você executou o programa. Este arquivo pode ser usado para análise dos sinais. Note que as contagens são dadas em termos de canais de ADC, a serem convertidas por você em volts, por exemplo.
