# ล่าสุด ตามเวลา commit 11/7/2018


## Feature 

  - ค้นหาไฟล์ TBT,BRF,TXT ใน SD Card ได้
  - เปิดอ่านไฟล์ TBT 
    -  อ่านไฟล์ sector ถัดไป Charator>4096 
    - เลื่อนบรรทัด ไป/กลับ (ไม่เกิน 4096 ตัวอักษร)
  - โหมดบลูทูธ
    - เชื่อมต่อกับอุปกรณ์ android ทำงานเป็น navigator 
    - Read text ด้วย Talkback และแสดงผ่านจุดเบรลล์
    - Keyboard Braille  
    - โหมดพิมพ์ เป็นแป้นพิมพ์อักษรเบรลล์
## ที่กำลังทำ 
  - บันทึกไฟล์ที่พิมพ์ในโหมด notepad
## ปัญหา 
  - # จน

## ปัญหาที่แก้ไขแล้ว 
  - ปรับความเร็ว Serial3  เป็น 115200
  - แก้อาการค้างเมื่อค้นหาไฟล์
  - ย้อนกลับจากโหมดอ่านไม่ได้ ต้องรีเซ็ท
  - ยังเปิดและดูไฟล์ในโฟลเดอร์ไม่ได้
  - ไม่สามารถปรับ baud rate จาก 9600 115200 ขณะรันโปรแกรม ใน STM32 ได้
  - อ่านไฟล์ช้ามากๆ
  - Refresh dot ช้าา
#### Serial config 
```c
  //9600 connect to bluetooth module
  UART4_Configuration(); 
  //115200 connect with keyboard
  USART2_Configuration(); 
  //115200 defualt serial
  USART1_Configuration();
  //115200 connect to ch376 (sd card module)
  USART3_Configuration();
```

#### Output Function
```c
   //ส่ง Braille unicode ไปแสดงที่จุดเบรลล์
   printDot(brailleUnicodeByte, sizeof(brailleUnicodeByte)); 
   // ส่งสตริงไปแสดงที่จุดเบรลล์
   stringToUnicodeAndSendToDisplay("String here"); 
```

#### Keyboard Input 
```c
    if (USART_GetITStatus(USART2, USART_IT_RXNE)) {
    //----------------------------- uart to key--------------------------------
    uart2Buffer = USART_ReceiveData(USART2);                                //-
    if (uart2Buffer == 0xff && SeeHead == 0) {                              //-
      SeeHead = 1;                                                          //-
      countKey = 0;                                                         //-
    }                                                                       //-

    if (countKey >= 4 && countKey <= 6) {                                   //-
      bufferKey3digit[countKey - 4] = uart2Buffer;                          //-
    }
    if (countKey == 2) //checkKeyError
    {
      checkKeyError = uart2Buffer;
    }
    countKey++;
    // ---------------------------- end uart to key ----------------------------
  }
  if (countKey >= maxData) { //Recieve & checking key
    seeHead = 0;
    printf("See key %x,%d,%x\r\n", bufferKey3digit[0], bufferKey3digit[1], bufferKey3digit[2]);
    //printf("checkKey :%x\r\n",checkKeyError);
    if (checkKeyError == 0xff) { //check error key
      //printf("Key Error");
      countKey = 0;
      SeeHead = 0;
    }
     if (bufferKey3digit[1] != 0 || bufferKey3digit[2] != 0) {
      // ---------------------------- to key code -----------------------------
      if (bufferKey3digit[2] == 1 || bufferKey3digit[2] == 0x20) { // check key array mapping to 'keyCode'
        keyCode = 40; // arrow down
      }
     }
  }
```
#### Mode (in void main)
###### Bluetooth Mode
```c
  notepad();
```
###### Bluetooth Mode
```c
   while (mode == 5) { // while in bluetooth mode
      BluetoothMode(); // 
      // keyboardMode();
      if (becon == 0) { 
        menu_s();
      }
      else {
        keyboardMode();  //connected use keyboard for bluetooth
      }
    }
```
###### Read Mode
```c
    while (mode == 3 && openFileStatus == 0) { //if enter to mode 3 search file and display
      searchFile();
    }
    while (mode == 3 && openFileStatus == 1) { //if open file 
      ReadFile();
    }
```




