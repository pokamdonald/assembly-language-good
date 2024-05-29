# assembly-language-good
SECURE DOOR SYSTEM


include emu8086.inc
.MODEL SMALL

.DATA   
        MAX_USERS EQU 10
        HEADER_MSG DB '==== Secure Home Access System ====$'
        PROMPT_ID_MSG DB 13, 10, 'Please Enter your ID:$'
        PROMPT_PASS_MSG DB 13, 10, 'Please Enter your Password:$'
        ERROR_ID_MSG DB 13, 10, 'ERROR: ID Not Found!$'
        ERROR_PASS_MSG DB 13, 10, 'ERROR: Incorrect Password! Access Denied.$'
        SUCCESS_MSG DB 13, 10, 'Access Granted! Welcome Home.$'
        ERROR_PASS_LONG_MSG DB 13, 10, 'ERROR: Password Too Long!$'
        ENTERED_ID DW 1 DUP(?), 0
        ENTERED_PASS DB 1 DUP(?)
        ID_SIZE = $ - ENTERED_ID
        PASS_SIZE = $ - ENTERED_PASS

        ; Updated user IDs and passwords
        USER_IDS DW 'X123', 'Y456', 'Z789', 'A321', 'B654', 'C987', 'D159', 'E753', 'F852', 'G951'
        USER_PASSWORDS DB 5, 8, 12, 20, 25, 30, 35, 40, 45, 50

.CODE
MAIN PROC
            ; Initialize data segment
            MOV AX, @DATA
            MOV DS, AX
            MOV AX, 0000H

DisplayHeader:
            ; Display the header
            LEA DX, HEADER_MSG
            MOV AH, 09H
            INT 21H

PromptForID:
            ; Prompt for ID
            LEA DX, PROMPT_ID_MSG
            MOV AH, 09H
            INT 21H
            
GetID:
            ; Input ID
            MOV BX, 0
            MOV DX, 0
            LEA DI, ENTERED_ID
            MOV DX, ID_SIZE
            CALL get_string
            
ValidateID:
            ; Check if ID exists
            MOV BL, 0
            MOV SI, 0

CheckIDLoop:
            MOV AX, USER_IDS[SI]
            MOV DX, ENTERED_ID
            CMP DX, AX
            JE PromptForPassword
            INC BL
            ADD SI, 4
            CMP BL, MAX_USERS
            JB CheckIDLoop
            
DisplayIDError:
            ; ID not found error message
            LEA DX, ERROR_ID_MSG
            MOV AH, 09H
            INT 21H
            JMP PromptForID
             
PromptForPassword:
            ; Prompt for password
            LEA DX, PROMPT_PASS_MSG
            MOV AH, 09H
            INT 21H
            
GetPassword:
            ; Input password
            CALL scan_num
            CMP CL, 0FH
            JAE PasswordTooLong
            MOV BH, 00H
            MOV DL, USER_PASSWORDS[BX]
            CMP CL, DL
            JE PasswordCorrect
            
DisplayPasswordError:
            ; Incorrect password message
            LEA DX, ERROR_PASS_MSG
            MOV AH, 09H
            INT 21H
            JMP PromptForID
            
PasswordCorrect:
            ; Correct password message and sound
            LEA DX, SUCCESS_MSG
            MOV AH, 09H
            INT 21H
            
            ; Beep the speaker
            MOV AX, 0E07H
            MOV BH, 00H
            MOV BL, 07H
            INT 10H

            JMP TerminateProgram

PasswordTooLong:
            ; Password too long error message
            LEA DX, ERROR_PASS_LONG_MSG
            MOV AH, 09H
            INT 21H
            JMP PromptForPassword

DEFINE_SCAN_NUM
DEFINE_GET_STRING

TerminateProgram:
            ; End program
            MOV AX, 4C00H
            INT 21H
            END MAIN
