# projeto2semestree
Este projeto visa simular um banco onde existem funções que apertando cada número podem executar as seguintes ações: criar nova conta, remover conta, debitar, transferir ou sacar dinheiro das contas ou listar os clientes.
LCD_RS equ P1.3
LCD_EN equ P1.2
LED_FECHADO equ P2.1 ; LED vermelho (fechado)
LED_ABERTO equ P2.0  ; LED verde (aberto)

org 0000h
    LJMP INICIO

org 0030h
INICIO:
	 CLR LCD_EN
	 SJMP $ ; Configura a senha padrão
    MOV 40H, #'0'
    MOV 41H, #'0'
    MOV 42H, #'0'
    MOV 43H, #'0'
    ACALL inicializaLCD
    ACALL EXIBE_MSG
    ACALL CONFIGURA_LEDS ; Inicializa LEDs (fechadura inicialmente fechada)

CONFIGURA_LEDS:
    ; Inicializa o estado dos LEDs
    SETB LED_FECHADO  ; Acende LED Vermelho (Fechado)
    CLR LED_ABERTO    ; Apaga LED Verde (Aberto)
    RET

EXIBE_MSG:
    MOV A, #01h
    ACALL posicionaCursor
    MOV A, #'D'
    ACALL escreveCaractere
    MOV A, #'I'
    ACALL escreveCaractere
    MOV A, #'G'
    ACALL escreveCaractere
    MOV A, #'I'
    ACALL escreveCaractere
    MOV A, #'T'
    ACALL escreveCaractere
    MOV A, #'E'
    ACALL escreveCaractere
    MOV A, #' '
    ACALL escreveCaractere
    MOV A, #'A'
    ACALL escreveCaractere
    MOV A, #' '
    ACALL escreveCaractere
    MOV A, #'S'
    ACALL escreveCaractere
    MOV A, #'E'
    ACALL escreveCaractere
    MOV A, #'N'
    ACALL escreveCaractere
    MOV A, #'H'
    ACALL escreveCaractere
    MOV A, #'A'
    ACALL escreveCaractere
    RET
FULL: ; Função para aguardar `#` ou `*` quando a memória está cheia
    ACALL leituraTeclado
    JNB F0, FULL  ; Aguarda até que uma tecla seja pressionada
    MOV A, #40H   ; Carrega o endereço base da memória onde a senha está armazenada
    ADD A, R0     ; Adiciona o valor de R0 ao endereço para obter o valor da tecla
    MOV R0, A     ; Move o valor resultante para R0
    MOV A, @R0    ; Carrega o valor da memória no acumulador
    CLR F0        ; Limpa a flag F0
    CJNE A, #42, HASH ; Se o valor lido não for `*` (42), vá para HASH
    CALL VERIFICA_SENHA ; Se for `*`, chama a função que verifica a senha
    RET

HASH: 
    CJNE A, #35, FULL ; Se o valor lido não for `#` (35), continue aguardando
    CALL REDIFINE_SENHA ; Se for `#`, chama a função que redefine a senha
    RET
VERIFICA_SENHA:
    ; Função para verificar a senha digitada com a senha armazenada
    MOV R1, #40H ; Inicia o ponteiro na senha armazenada
    MOV R2, #60H ; Inicia o ponteiro na senha digitada
    MOV A, @R2
    CJNE A, @R1, ERRO
    INC R1
    INC R2
    MOV A, @R2
    CJNE A, @R1, ERRO
    INC R1
    INC R2
    MOV A, @R2
    CJNE A, @R1, ERRO
    INC R1
    INC R2
    MOV A, @R2
    CJNE A, @R1, ERRO
    ; Senha correta
    ACALL ABRE_FECHADURA
    RET

ERRO:
    ; Exibe mensagem de erro e mantém LED vermelho
    ACALL MSG_ERRADA
    SETB LED_FECHADO   ; Acende o LED vermelho (fechado)
    CLR LED_ABERTO     ; Apaga o LED verde (aberto)
    RET


REDIFINE_SENHA:
    ; Função para redefinir senha
    MOV A, #'N'
    ACALL escreveCaractere
    MOV A, #'O'
    ACALL escreveCaractere
    MOV A, #'V'
    ACALL escreveCaractere
    MOV A, #'A'
    ACALL escreveCaractere
    ACALL AGUARDA_NOVA_SENHA
    RET

AGUARDA_NOVA_SENHA:
    ; Espera e salva nova senha
    ACALL leituraTeclado
    MOV 40H, A
    ACALL leituraTeclado
    MOV 41H, A
    ACALL leituraTeclado
    MOV 42H, A
    ACALL leituraTeclado
    MOV 43H, A
    ; Exibe mensagem "Senha definida"
    ACALL MSG_SENHA_DEFINIDA
    RET

MSG_SENHA_DEFINIDA:
    ; Exibe a mensagem de "Senha definida"
    MOV A, #01H
    ACALL posicionaCursor
    MOV A, #'S'
    ACALL escreveCaractere
    MOV A, #'E'
    ACALL escreveCaractere
    MOV A, #'N'
    ACALL escreveCaractere
    MOV A, #'H'
    ACALL escreveCaractere
    MOV A, #'A'
    ACALL escreveCaractere
    MOV A, #' '
    ACALL escreveCaractere
    MOV A, #'D'
    ACALL escreveCaractere
    MOV A, #'E'
    ACALL escreveCaractere
    MOV A, #'F'
    ACALL escreveCaractere
    MOV A, #'I'
    ACALL escreveCaractere
    MOV A, #'N'
    ACALL escreveCaractere
    MOV A, #'I'
    ACALL escreveCaractere
    MOV A, #'D'
    ACALL escreveCaractere
    MOV A, #'A'
    ACALL escreveCaractere
    RET

MSG_ERRADA:
    ; Exibe mensagem de erro
    MOV A, #01H
    ACALL posicionaCursor
    MOV A, #'E'
    ACALL escreveCaractere
    MOV A, #'R'
    ACALL escreveCaractere
    MOV A, #'R'
    ACALL escreveCaractere
    MOV A, #'O'
    ACALL escreveCaractere
    RET


ABRE_FECHADURA:
    ; Abre a fechadura e acende o LED verde
    CLR LED_FECHADO  ; Apaga o LED vermelho
    SETB LED_ABERTO   ; Acende o LED verde
    ACALL MSG_LIBERADO
    RET

MSG_LIBERADO:
    ; Exibe mensagem de sucesso (fechadura liberada)
    MOV A, #01H
    ACALL posicionaCursor
    MOV A, #'L'
    ACALL escreveCaractere
    MOV A, #'I'
    ACALL escreveCaractere
    MOV A, #'B'
    ACALL escreveCaractere
    MOV A, #'E'
    ACALL escreveCaractere
    MOV A, #'R'
    ACALL escreveCaractere
    MOV A, #'A'
    ACALL escreveCaractere
    MOV A, #'D'
    ACALL escreveCaractere
    MOV A, #'O'
    ACALL escreveCaractere
    RET

escreveCaractere:
    SETB LCD_RS
    MOV P1.7, C
    CALL delay
    CLR LCD_RS
    RET

posicionaCursor:
    MOV P1.7, #1
    RET

delay:
    MOV R7, #120
    DJNZ R7, $
    RET

leituraTeclado:
    MOV R0, #0
    MOV P0, #0FFH
    CLR P0.0
    CALL varreduraColunas
    JB F0, fimLeitura
    RET

varreduraColunas:
    JNB P0.4, teclaDetectada
    INC R0
    RET

teclaDetectada:
    SETB F0
    RET

inicializaLCD:
	  CLR P1.2
	  SETB P1.2
    CLR LCD_RS
    CLR P1.7
    SETB LCD_EN
    ACALL delay
     
RET
  
