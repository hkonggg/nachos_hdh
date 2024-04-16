// exception.cc 
//	Entry point into the Nachos kernel from user programs.
//	There are two kinds of things that can cause control to
//	transfer back to here from user code:
//
//	syscall -- The user code explicitly requests to call a procedure
//	in the Nachos kernel.  Right now, the only function we support is
//	"Halt".
//
//	exceptions -- The user code does something that the CPU can't handle.
//	For instance, accessing memory that doesn't exist, arithmetic errors,
//	etc.  
//
//	Interrupts (which can also cause control to transfer from user
//	code into the Nachos kernel) are handled elsewhere.
//
// For now, this only handles the Halt() system call.
// Everything else core dumps.
//
// Copyright (c) 1992-1993 The Regents of the University of California.
// All rights reserved.  See copyright.h for copyright notice and limitation 
// of liability and disclaimer of warranty provisions.

#include "copyright.h"
#include "system.h"
#include "syscall.h"
#include "thread.h"
#include "synch.h"
#include "addrspace.h"
#include "utility.h"


#define MAX_SIZE 100
#define MAX_FILE_LEN 32
#define MAX_LEN 255


//----------------------------------------------------------------------
// ExceptionHandler
// 	Entry point into the Nachos kernel.  Called when a user program
//	is executing, and either does a syscall, or generates an addressing
//	or arithmetic exception.
//
// 	For system calls, the following is the calling convention:
//
// 	system call code -- r2
//		arg1 -- r4
//		arg2 -- r5
//		arg3 -- r6
//		arg4 -- r7
//
//	The result of the system call, if any, must be put back into r2. 
//
// And don't forget to increment the pc before returning. (Or else you'll
// loop making the same system call forever!
//
//	"which" is the kind of exception.  The list of possible exceptions 
//	are in machine.h.
//----------------------------------------------------------------------
void AdvancePC() {
    int pcAfter = machine->registers[NextPCReg] + 4;
    machine->registers[PrevPCReg] = machine->registers[PCReg];
    machine->registers[PCReg] = machine->registers[NextPCReg];
    machine->registers[NextPCReg] = pcAfter;
}

int power(int x, int n) {
    int result = 1;
    for (int i = 0; i < n; i++) 
        result *= x;
    return result;
} 

char* User2System(int virtAddr,int limit)
{
    int i;// index
    int oneChar;
    char* kernelBuf = NULL;
    kernelBuf = new char[limit +1];//need for terminal string
    if (kernelBuf == NULL)
    return kernelBuf;
    memset(kernelBuf,0,limit+1);
    //printf("\n Filename u2s:");
    for (i = 0 ; i < limit ;i++)
    {
        machine->ReadMem(virtAddr+i,1,&oneChar);
        kernelBuf[i] = (char)oneChar;
        //printf("%c",kernelBuf[i]);
        if (oneChar == 0)
            break;
    }
    return kernelBuf;
}

int System2User(int virtAddr,int len,char* buffer)
{
    if (len < 0) return -1;
    if (len == 0)return len;
    int i = 0;
    int oneChar = 0 ;
    do{
        oneChar= (int) buffer[i];
        machine->WriteMem(virtAddr+i,1,oneChar);
        i ++;
    }while(i < len && oneChar != 0);
    return i;
}

void syscallSub(){
    int op1 = machine->ReadRegister(4);
    int op2 = machine->ReadRegister(5);
    int result = op1 - op2;
    machine->WriteRegister(2, result);
    interrupt->Halt();
}

void syscallReadInt(){
    //Input: None
    //Output: Trả về số nguyên
    //Chức năng: Đọc số nguyên từ màn hình console

    //Khai báo mảng kiểu char để chứa dữ liệu nhập từ console
    char* buffer = new char[MAX_LEN];

    //Gọi hàm synchconsole để đọc buffer, và trả về số bytes đọc được
    int numberBytesRead = synchconsole->Read(buffer, MAX_LEN); 

    //Kiểm tra xem số bytes đọc được có âm không
    if(numberBytesRead < 0){
        printf("\n Cannot read the integer\n");
        delete[]buffer; 
        return;
    }

    //Xử lý chuỗi nhập vào ---> Số nguyên
    int numberIndex = 1;
    char sig = buffer[0];
    if(sig != '-'){
        numberIndex = 0;
    }

    int result = 0;
    for(int i = numberIndex; i < numberBytesRead; i++){
        if(buffer[i] >= '0' && buffer[i] <= '9'){
            result = result + (buffer[i] - '0') * power(10, numberBytesRead - i - 1);
        }
        else{ //Có ký tự khác [0, 9] tự động trả về giá trị 0
            printf("\nInvalid digit, Default will be zero(0)\n");
            result = 0;
            machine->WriteRegister(2, result);
            delete[]buffer; 
            return;
         }
    }

    //Ghi kết quả vừa đọc được vào thanh ghi số 2
    if(sig == '-'){
        result = result * (-1);
    }

    machine->WriteRegister(2, result);

    //Deallocate các pointer cấp phát động
    delete[]buffer; 
    return;
}

void syscallPrintInt(){
    //Input: Một số nguyên
    //Output: None
    //Chức năng: In chuỗi kí tự số nguyên ra màn hình console

    //Đọc giá trị số nguyên ở thanh ghi số 4
    int number = machine->ReadRegister(4);

    //Kiểm tra số nhập được có bằng 0
    if(number == 0){
        synchconsole->Write("0", 1);
        return;
    }
    
    //Xử lý chuỗi số nguyên --> chuỗi ký tự(char*)
    bool isNegative = false;
    int numberSize = 0;

    if(number < 0){
        isNegative = true;
        number = number * (-1);
    } 

    int temp = number;
    while(temp > 0){
        numberSize++;
        temp = temp / 10;
    } //Đọc độ dài của số nguyên đó

    char* buffer = new char[MAX_SIZE + numberSize];
    temp = number; 
   
	if (isNegative) {
		buffer[0] = '-';
		numberSize = numberSize + 1;
	}

	for (int i = numberSize - 1; i >= 0; i--) {
		if (isNegative && i == 0) break;
		int k = temp % 10;
		buffer[i] = char(k) + '0';
		temp = temp / 10;
	}

	buffer[numberSize] = 0;

    //In ra màn hình console với số lượng size của số nguyên đó
    synchconsole->Write(buffer, numberSize);
    
    //Deallocate các pointer được cấp phát động
    delete[]buffer;
    return;
    
}

void syscallReadChar(){
    //Input: None
    //Output: ký tự(char)
    //Chức năng: Đọc một ký tự ở màn hình console

    //Khai báo mảng char để chứa dữ liệu nhập vào
    int maxBytes = 255;
    char* buffer = new char[maxBytes];

    //Gọi hàm synchconsole để đọc buffer và trả về số bytes đọc được
    int numBytesRead = synchconsole->Read(buffer, maxBytes);

    //Số bytes đọc được lớn hơn 1 ==> Chuỗi ký tử ==> Trả về ERROR
    if(numBytesRead > 1){
        printf("Only 1 character input");
        DEBUG('a', "\nERROR: Only 1 character input!!!");
        machine->WriteRegister(2, 0);
    }

    //Số bytes đọc được bằng 0 ==> Ký tự rỗng ==> Trả về ERROR
    else if(numBytesRead == 0){
        printf("Empty character");
        DEBUG('a', "\nERROR: Empty character!!!");
        machine->WriteRegister(2, 0);
    }

    //Ghi kí tự vào thanh ghi số 2
    else{
        char c = buffer[0];
        machine->WriteRegister(2, c);
    }

    //Deallocate các ptr cấp phát động
    delete[] buffer;
    return;
}

void syscallPrintChar(){
    //Input: Ký tự(char)
    //Output: None
    //Chức năng: In ra một ký tự ở màn hình console
    
    //Đọc ký tự từ thanh ghi số 4
    char c = (char)(machine->ReadRegister(4));

    //In ra ký tự đó từ địa chỉ của nó, in 1 byte
    synchconsole->Write(&c, 1);
    return;
}


void syscallReadString(){
    //Input: Mảng char(char* or char[])
    //Output: None
    //Chức năng: Đọc 1 chuỗi ký tự từ console

    int virtAddr, len;
    char* buffer;
    //Lấy địa chỉ tham số buffer truyền vào thừ thanh ghi 4
    virtAddr = machine->ReadRegister(4);

    //Khai báo mảng char với độ dài tối đa(255 bytes)
    buffer = new char[MAX_LEN];

    //Copy chuỗi từ vùng nhớ User Space sang Kernel Space
    buffer = User2System(virtAddr, MAX_LEN);
    
    //Gọi hàm synchconsole để đọc chuỗi vào buffer, trả về số bytes đọc được
    int numberBytesRead = synchconsole->Read(buffer, MAX_LEN);

    //Kiểm tra số bytes đọc được có âm không
    if(numberBytesRead < 0){
        delete[] buffer;
        printf("Empty string");
        return;
    }

    //Copy chuỗi từ vùng nhớ Kernel Space về User Space
    System2User(virtAddr, MAX_LEN, buffer);

    //Deallocate các ptr cấp phát động
    delete[]buffer;
    return;
}

void syscallPrintString(){
    //Input: Mảng char(char* or char[])
    //Output: None
    //Chức năng: In ra 1 chuỗi ký tự ở màn hình console
    int virtAddr;
    int maxBytes = 255;
    char* buffer;
    //Lấy địa chỉ của tham số buffer từ thanh ghi số 4
    virtAddr = machine->ReadRegister(4);
    //Copy chuỗi từ vùng nhớ User Space sang Kernel Space với bộ đếm dài 255 bytes
    buffer = User2System(virtAddr, maxBytes);

    //Lấy độ dài thực sự của chuỗi ký tự
    int len = 0;
    while(buffer[len] != 0) len++;

    //Gọi hàm synchconsole để in chuỗi
    synchconsole->Write(buffer, len);

    //Deallocate các ptr cấp phát động
    delete[] buffer;

    return;
}

void syscallCreate(){
    //Input: Tên file muốn tạo(Địa chỉ của nó)
    //Output: -1 = Lỗi, 0 = Thành công
    //Chức năng: Tạo một file vào hệ thống

    int virtAddr;
    char* filename;

    DEBUG('a',"\n SC_Create call ...\n");
    DEBUG('a',"\n Reading virtual address of filename\n");
    // Kiểm tra các exception

    //Đọc địa chỉ của file từ thanh ghi số 4
    virtAddr = machine->ReadRegister(4);
    DEBUG ('a',"\n Reading filename.\n");

    //Copy từ User Space sang Kernel Space, với độ dài là 32 bytes
    filename = User2System(virtAddr,MAX_FILE_LEN + 1); // MaxFileLength là = 32

    //Không đủ ô nhớ để tạo file trong Kernel
    if (filename == NULL)
    {
        printf("\n Not enough memory in system\n");
        DEBUG('a',"\n Not enough memory in system\n");
        machine->WriteRegister(2,-1); // trả về lỗi cho chương trình người dùng
        delete filename;
        return;
    }

    //Tên file rỗng ==> Không tạo được
    if(filename[0] == 0){
        printf("\n =====> File name cant be empty\n");
        machine->WriteRegister(2, -1);
        delete filename;
        return;
    }

    DEBUG('a',"\n Finish reading filename.\n");

    //Tạo file bằng hàm Create của file system
    if (!fileSystem->Create(filename,0)){
        printf("\n Error create file '%s'",filename);
        printf("\n");
        machine->WriteRegister(2,-1);
        delete filename;

        return;
    }

    machine->WriteRegister(2,0); // Trả về cho chương trình người dùng ==> Tạo file thành công
    delete filename;
    return;
}

void syscallOpen(){
    //Input: Tên file cần mở, chức năng mở
    //Output: Trả về ID của file, -1 nếu bị lỗi
    //Chức năng: Mở một file để đọc

    //Lấy địa chỉ của tham số name từ thanh ghi số 4
    int virtAddr = machine->ReadRegister(4);
    //Lấy cách mở file đó tại thanh ghi số 5
    int type = machine->ReadRegister(5);


    char* filename;
    //Copy filename từ User Space sang Kernel Space, với bộ đếm file name(32 bytes)
    filename = User2System(virtAddr, MAX_FILE_LEN);

    //Tìm 1 slot trống trong bảng mô tả file [2, 10] (2 slot đầu cho console input và output)
    int fileSlot = fileSystem->FindFreeSlot();
    
    //Xử lý khi còn slot trống
    if(fileSlot != -1){
        if(type == 0 || type == 1){ //Xử lý các type đọc và ghi file hoặc chỉ ghi file
            if((fileSystem->openf[fileSlot] = fileSystem->Open(filename, type)) != NULL){ //Mở file thành công
                fileSystem->openf[fileSlot]->setName(filename); //Lưu tên của file đó cho thuận tiện việc xóa file
                machine->WriteRegister(2, fileSlot); //Trả về ID của file đó(OpenFileID)
            }
        }
        
        //Nếu type thuộc đặc tả console input
        else if(type == 2){
            machine->WriteRegister(2, 0); //Trả về ID 0 trong bảng mô tả file
        }

        //Nếu type thuộc đặc tả console output
        else{
            machine->WriteRegister(2, 1); //Trả về ID 1 trong bảng mô tả file
        }

        delete[] filename;
        return;
    }

    //Mở file thất bải trả về -1
    machine->WriteRegister(2, -1);
    delete[] filename;
    return;
}

void syscallClose(){
    //Input: ID của file(OpenFileID)
    //Output: 0 -- Thành công, -1 -- Thất bại
    //Chức năng: Đóng một file trong hệ thống bằng ID của nó

    //Lấy ID của file tại thanh ghi số 4
    int ID = machine->ReadRegister(4);

    //Kiểm tra xem ID lấy được có nằm trong bảng mô tả file
    if(ID >= 0 && ID <= 9){
        if(fileSystem->openf[ID] != NULL){
            delete fileSystem->openf[ID];
            fileSystem->openf[ID] = NULL; //Reset file slot thành slot trống 
            machine->WriteRegister(2, 0); //Trả về 0 ==> Đóng file thành công
            return;
        }

        machine->WriteRegister(2, -1); //Trả về -1 ==> Đóng file thất bại
        return;
    }
    return;
}

void syscallRead(){
    //Input: buffer(char*), độ dài buffer(int), ID file cần đọc(OpenFileID)
    //Output: 0 ==> Đọc thành công, -1 ==> Không đọc được file
    //Chức năng: Đọc 1 file trong hệ thống

    //Lấy địa chỉ của buffer tại thanh ghi số 4
    int virtAdrr = machine->ReadRegister(4);
    //Lấy độ dài của buffer đó tại thanh ghi số 5
    int charCount = machine->ReadRegister(5);
    //Lấy ID của file tại thanh ghi số 6
    int ID = machine->ReadRegister(6);

    int oldPos, newPos;
    char* buffer;
    //Kiểm tra xem ID của file có nằm ngoài bảng mô tải file hoặc là slot đó có NULL không
    if(ID < 0 || ID > 9 || fileSystem->openf[ID] == NULL){
        printf("\n ===> File is unavailable\n");
        machine->WriteRegister(2, -1); //Trả về -1, file không tồn tại
        return;
    }

    //Nếu ID file là một console output
    else if(fileSystem->openf[ID] -> type == 3){ 
        printf("\n ===> Unavailable to read stdout file\n");
        machine->WriteRegister(2, -1); //Trả về -1, vì không đọc được stdout file
        return;
    }

    //Xử lý đọc file
    oldPos = fileSystem->openf[ID]->GetCurrentPos(); //Lấy vị trí đầu tiên của file
    buffer = User2System(virtAdrr, charCount); //Copy chuỗi từ User Space sang Kernel Space

    if(fileSystem->openf[ID]->type == 2){ //Nếu ID file là một console input
        int numberBytesRead = synchconsole->Read(buffer, charCount); //Sử dụng hàm synchconsole để đọc buffer, trả về số bytes đọc được
        System2User(virtAdrr, numberBytesRead, buffer); // Copy chuỗi từ Kernel Space vê User Space

        if(numberBytesRead < MAX_LEN){
            machine->WriteRegister(2, numberBytesRead); //Số bytes đọc được < 255 bytes(Max bytes của file) ==> Ghi vào thanh ghi số 2
        }
        else{
            machine->WriteRegister(2, -1); //Nếu lơn hơn 255 bytes ==> Trả về -1, không ghi được vào file
        }

        delete[] buffer;
        return;
    } 
    
    //Xử lý đọc file bình thương(.txt ..etc..)
    if((fileSystem->openf[ID]->Read(buffer, charCount)) > 0){
        newPos = fileSystem->openf[ID]->GetCurrentPos(); //Lấy vị trí hiện tại của con trỏ file trỏ đến
        System2User(virtAdrr, newPos - oldPos, buffer); //Copy chuỗi từ Kernel Space về User Space, số bytes thực sự = newPos - oldPos

        if((newPos - oldPos) < MAX_LEN){
            //Kiểm tra số bytes đọc được < 255 ==> Trả về số bytes thực sự đó, đọc thành công
            machine->WriteRegister(2, newPos - oldPos); 
        }
        else {
            //Trả về -1, nếu lớn hơn 255 bytes vì không đủ memory để đọc
            machine->WriteRegister(2, -1);
        }
    }

    else{
        //Trả về -2 nếu như file rỗng
        printf("\n ===> Empty file\n");
        machine->WriteRegister(2, -2);
    }
    delete[]buffer;
    return;
}

void syscallWrite(){
    //Input: buffer(char*), độ dài muốn write(int), ID của file(OpenFileID)
    //Output: -1 -- Lỗi, Số bytes write thành công -- Thành công
    //Chức năng: Viết dữ liệu buffer vào 1 file trong hệ thống

    //Lấy địa chỉ của buffer tại thanh ghi số 4
    int virtAdrr = machine->ReadRegister(4);
     //Lấy độ dài muốn viết tại thanh ghi số 5
    int charCount = machine->ReadRegister(5);
    //Lấy ID của file tại thanh ghi số 6
    int ID = machine->ReadRegister(6);

    int oldPos, newPos;
    char* buffer;

    //Kiểm tra xem ID của file có nằm ngoài bảng mô tải file hoặc là slot đó có NULL không
    if(ID < 0 || ID > 9 || fileSystem->openf[ID] == NULL){
        printf("\n ===> File is unavailable\n");
        machine->WriteRegister(2, -1); //Trả về -1, vì file không tồn tại hoặc nằm ngoài bảng mô tả file
        return;
    }
    
    //Kiểm tra xem file có phải là Read-Only file hoặc là file console input
    if(fileSystem->openf[ID]->type == 1 || fileSystem->openf[ID]->type == 2){
        printf("\n ==> Unavailable to write into stdin and read-only file\n");
        machine->WriteRegister(2, -1); //Trả về -1
        return;
    } //error with type to write

    //Xử lý file
    oldPos = fileSystem->openf[ID]->GetCurrentPos(); //Lấy vị trí đầu của file 
    buffer = User2System(virtAdrr, charCount); //Copy chuỗi từ User Space sang Kernel Space

    //Kiểm tra xem đây có phải là read&write file không
    if(fileSystem->openf[ID]->type == 0){
        if((fileSystem->openf[ID]->Write(buffer, charCount)) > 0){
            newPos = fileSystem->openf[ID]->GetCurrentPos(); //Lấy vị trí hiện tại mà con trỏ file trỏ đến
            machine->WriteRegister(2, newPos - oldPos); //Ghi số bytes write thực sự vào thanh ghi số 2
            delete[] buffer;
            return;
        }
    }

    if(fileSystem->openf[ID]->type == 3){ //Nếu file là một console output
        int idx = 0;
        while(buffer[idx] != 0){
            synchconsole->Write(buffer + idx, 1); //Write từng kí tự nhập vào tự console
            idx++;
        }
        synchconsole->Write(buffer + idx, 1); //Write kí tự "\n"
        machine->WriteRegister(2, idx - 1); //Trả về số bytes write vào
        delete[] buffer;
        return;
    }

    return;
}


void syscallDelete(){
   //Iput: Tên file cần xóa
   //Output: -1 -- Lỗi, 0 -- Thành công
   //Chức năng: Xóa 1 file trong hệ thống

   //Lấy địa chỉ của file name tại thanh ghi số 4
   int virtAddr = machine->ReadRegister(4);

   char* name;
   name = User2System(virtAddr, MAX_LEN + 1); //Copy chuỗi từ User Space sang Kernel Space


   if(name == NULL){ //Kiểm tra xem tên file nhập vào rỗng không
        printf("\n ===> Not enough memory in system\n");
        DEBUG('a', "\nNot enough memory in system\n");
        machine->WriteRegister(2, -1); //Trả về -1, vì không tìm được file
        delete[] name;
        return;
   } 

    //Tìm ID của file trong bảng mô tả file thông qua tên file vừa nhập
   for(int i = 2; i < 10; i++){
        if(fileSystem->openf[i] != NULL){
            if(strcmp(name, fileSystem->openf[i]->getName()) == 0){
                printf("\n File is currently opened, cannot be deleted\n");
                machine->WriteRegister(2, -1); //Trả về -1, vì file đang mở trong hệ thống
                delete[] name;
                return;
            }
        }
   }

   if(!fileSystem->Remove(name)){ //Kiểm tra file có tồn tại để xóa không
        printf("\n ==> File is not existed\n");
        machine->WriteRegister(2, -1);
        delete[] name;
        return;
   }
   machine->WriteRegister(2, 0); //Trả về 0, thành công xóa file đó khỏi hệ thống
   delete[] name;
   return;
}

void syscallReadFloat() {
    //Input: None
    //Output: Trả về số thực
    //Chức năng: Đọc số thực từ màn hình console

    // Khai báo mảng kiểu char để chứa
    char* buffer = new char[MAX_SIZE];

    // Gọi hàm Read của SynchConsole
    int numberBytesRead = synchconsole->Read(buffer, MAX_SIZE);

    if(numberBytesRead < 0) {
        printf("\n Cannot read the float");
        delete[] buffer;
        return;
    }

    // Xử lý chuỗi nhập vào --> số thực
    int numberIndex = 1;
    char sig = buffer[0];
    float* result = new float;
    *result = 0.0;
    int decimalIndex = -1;
    bool decimalFound = false;

    if(sig != '-'){
        numberIndex = 0;
    }

    for(int i = numberIndex; i < numberBytesRead; i++) {
        if(buffer[i] >= '0' && buffer[i] <= '9') {
            *result = *result * 10 + (buffer[i] - '0');

            if (decimalFound)
                decimalIndex++;
        }
        else if (buffer[i] == '.' && !decimalFound) {
            decimalFound = true;
            decimalIndex = 0;
        }
        else {
            printf("\nInvalid digit. Default will be zero(0.0)");
            *result = 0.0;
            break;
        }
    }

    // Thêm dấu thập phân
    if (decimalFound)
        *result /= power(10, decimalIndex);

    // Ghi kết quả số thực vào thanh ghi số 2 trả về
    if(sig == '-')
        *result = *result * (-1);


    //Lấy giá trị địa chỉ chứa float đó tra về
    int* addr;
    addr = (int*)result; //Ép kiểu theo int* để trả về giá trị địa chỉ int cho thanh ghi
    
    machine->WriteRegister(2, *addr);
    delete[] buffer;
    delete result;
    return;
}

void syscallPrintFloat() {
    //Input: Số thực
    //Output: None
    //Chức năng: In số thực ra màn hình console

    int number = machine->ReadRegister(4);
    float* ifloat = (float*)&number; //Lấy địa chỉ int đó ép kiểu float* để lấy ra giá trị số thực

    if (*ifloat == 0) {
        synchconsole->Write("0.0", 1);
        return;
    }

    // Xử lý dấu và lấy giá trị tuyệt đối của số
    bool isNegative = false;
    if (*ifloat < 0) {
        isNegative = true;
        *ifloat = *ifloat * (-1);
    }

    // Lấy phần nguyên và phần thập phân của số
    int intPart = (int)(*ifloat);
    float floatPart = *ifloat - intPart;

    // Tạo một buffer lưu trữ chuỗi kết quả
    const int MAX_SIZEE = 32;
    char* buffer = new char[MAX_SIZEE];
    int index = 0;

    // Xử lý phần nguyên
    do {
        buffer[index++] = '0' + intPart % 10;
        intPart /= 10;
    } while (intPart > 0);

    // Nếu số là âm, thêm dấu '-' vào chuỗi kết quả
    if (isNegative){
        buffer[index++] = '-';

    }

    // Đảo chuỗi phần nguyên lại để đảm bảo đúng thứ tự
    for (int i = 0; i < index / 2; i++) {
        char temp = buffer[i];
        buffer[i] = buffer[index - i - 1];
        buffer[index - i - 1] = temp;
    }

    // Thêm dấu thập phân và phần thập phân nếu có
    buffer[index++] = '.';
    int decimalIndex = 0;
    do {
        floatPart *= 10;
        buffer[index++] = '0' + int(floatPart) % 10;
        decimalIndex++;
    } while (decimalIndex < 6 && floatPart != 0);

    // Kết thúc chuỗi kết quả
    buffer[index] = '\0';

    // Ghi chuỗi kết quả vào màn hình
    synchconsole->Write(buffer, index);
    delete[] buffer;
    return;
}

void syscallFloat_To_Buffer(){
    //Input: Mảng char(char* or char[])
    //Output: None
    //Chức năng: In ra 1 chuỗi ký tự ở màn hình console
    int virtAddr; int number;
    int maxBytes = 255;
    char* buffer;
    //Lấy địa chỉ của tham số buffer từ thanh ghi số 4
    virtAddr = machine->ReadRegister(4);
    number = machine->ReadRegister(5);
    //Copy chuỗi từ vùng nhớ User Space sang Kernel Space với bộ đếm dài 255 bytes
    buffer = User2System(virtAddr, maxBytes);


    float* ifloat = (float*)&number; //Lấy địa chỉ int đó ép kiểu float* để lấy ra giá trị số thực

    if (*ifloat == 0) {
        synchconsole->Write("0.0", 1);
        return;
    }

    // Xử lý dấu và lấy giá trị tuyệt đối của số
    bool isNegative = false;
    if (*ifloat < 0) {
        isNegative = true;
        *ifloat = *ifloat * (-1);
    }

    // Lấy phần nguyên và phần thập phân của số
    int intPart = (int)(*ifloat);
    float floatPart = *ifloat - intPart;

    int index = 0;

    // Xử lý phần nguyên
    do {
        buffer[index++] = '0' + intPart % 10;
        intPart /= 10;
    } while (intPart > 0);

    // Nếu số là âm, thêm dấu '-' vào chuỗi kết quả
    if (isNegative){
        buffer[index++] = '-';

    }

    // Đảo chuỗi phần nguyên lại để đảm bảo đúng thứ tự
    for (int i = 0; i < index / 2; i++) {
        char temp = buffer[i];
        buffer[i] = buffer[index - i - 1];
        buffer[index - i - 1] = temp;
    }

    // Thêm dấu thập phân và phần thập phân nếu có
    buffer[index++] = '.';
    int decimalIndex = 0;
    do {
        floatPart *= 10;
        buffer[index++] = '0' + int(floatPart) % 10;
        decimalIndex++;
    } while (decimalIndex < 6 && floatPart != 0);

    // Kết thúc chuỗi kết quả
    buffer[index] = '\0';


    //Lấy độ dài thực sự của chuỗi ký tự
    int len = 0;
    while(buffer[len] != 0) len++;

    System2User(virtAddr, maxBytes, buffer);
    machine->WriteRegister(2, len);

    //Deallocate các ptr cấp phát động
    delete[] buffer;
    return;
}

void syscallFloatCompare() {
    int a = machine->ReadRegister(4);
    int b = machine->ReadRegister(5);
    float* ifloat1 = (float*)&a; 
    float* ifloat2 = (float*)&b; 
    float num1 = *ifloat1;
    float num2 = *ifloat2;
    int result;
    if(num1 == num2) {
        result = 1;
    }
    else if(num1 > num2) {
        result = 2;
    }
    else {
        result = 3;
    }
    machine -> WriteRegister(2, result);
    return;
}


void ExceptionHandler_Exec() {
    int nameAdd = machine->ReadRegister(4);
    if (nameAdd == 0) {
        machine->WriteRegister(2, -1);
        return;
    }
    /*load file name from user to kernel*/
    char* name = User2System(nameAdd, 255);
    if (name == NULL) {
        printf("Không mở được file\n");
        machine->WriteRegister(2, -1); // Gán -1 vào thanh ghi r2
        return;
    }
    int res = pTab->ExecUpdate(name);
    machine->WriteRegister(2, res);
    
    delete [] name;
}
void ExceptionHandler_Join() {  
   //Đọc id của tiến trình cần Join từ thanh ghi r4.
    int id = machine->ReadRegister(4);
//Gọi thực hiện pTab->JoinUpdate(id) và lưu kết quả thực hiện của hàm vào thanh ghi r2. 
    int res = pTab->JoinUpdate(id);
    machine->WriteRegister(2, res);
}

void ExceptionHandler_Exit() { 
    //Đọc exitStatus từ thanh ghi r4 
    int exitStatus = machine->ReadRegister(4);
  //Gọi thực hiện pTab->ExitUpdate(exitStatus) và lưu kết quả thực hiện của hàm vào thanh ghi r2. 
    int res = pTab->ExitUpdate(exitStatus);
    machine->WriteRegister(2, res);
}
void ExceptionHandler_CreateSemaphore() {
    int nameAdd = machine->ReadRegister(4);
    int semval = machine->ReadRegister(5);
    if (nameAdd == 0 || semval < 0) {
        machine->WriteRegister(2, -1);
        return;
    }
    
    char* name = User2System(nameAdd, 255);
    int res = semTab->Create(name, semval);
    machine->WriteRegister(2, res);
    delete [] name;
}

void ExceptionHandler_Up(){
    int nameAdd = machine->ReadRegister(4);
    if (nameAdd == 0) {
        machine->WriteRegister(2, -1);
        return;
    }
    char* name = User2System(nameAdd, 255);
    if (!semTab->CheckSemaphore(name)) {
        printf("Semaphore %s không tồn tại trong bảng sTab\n", name);
        machine->WriteRegister(2, -1); // Ghi -1 vào thanh ghi r2
        return ;
    }
    int signalResult = semTab->Signal(name);
    machine->WriteRegister(2, signalResult);
delete [] name;
}
void ExceptionHandler_Down() {
    int nameAdd = machine->ReadRegister(4);
    if (nameAdd == 0) {
        machine->WriteRegister(2, -1);
        return;
    }
    /*use SEM_MAXNAMESIZE to get name*/
    char* name = User2System(nameAdd, 255);
    if (!semTab->CheckSemaphore(name)) {
        printf("Semaphore %s không tồn tại trong bảng sTab\n", name);
        machine->WriteRegister(2, -1); // Ghi -1 vào thanh ghi r2
        return ;
    }
    /*wait semaphore*/
    int res = semTab->Wait(name);
    machine->WriteRegister(2, res);
delete [] name;
}

void
ExceptionHandler(ExceptionType which)
{
    int type = machine->ReadRegister(2);

    switch(which) {
        case NoException:
            interrupt->Halt();
            return;
        case PageFaultException:
            DEBUG('a', "\n No valid translation found");
            printf("\n\n No valid translation found");
            interrupt->Halt();
            break;
        case ReadOnlyException:
            DEBUG('A',"\n Write attempted to page marked read-only");
            printf("\n\n Write attempted to page marke read-only");
            interrupt->Halt();
            break;
        case BusErrorException:
            DEBUG('a', "\n Translation resulted invalid physical address");
            printf("\n\n Translation resulted invalid physical address");
            interrupt->Halt();
            break;
        case AddressErrorException:
            DEBUG('a', "\n Unaligned reference or one that was beyond the end of the address space");
            printf("\n\n Unaligned reference or one that was beyond the end of the address space");
            interrupt->Halt();
            break;
        case IllegalInstrException:
            DEBUG('a', "\n Unimplemented or reserved instr.");
            printf("\n\n Unimplemented or rrvedion");
            interrupt->Halt();
            break;
        case NumExceptionTypes:
            DEBUG('a', "\n Number exception types");
            printf("\n\n Number exception types");
            interrupt->Halt();
            break;
        case SyscallException:
        {
            switch(type)
            {
                case SC_Halt:
                    DEBUG('a', "Shutdown, initiated by user program.\n");
                    interrupt->Halt();
                    return;
                case SC_Sub:
                    syscallSub();
                    break;
        
                case SC_ReadInt:
                    syscallReadInt();
                    break;

                case SC_PrintInt:
                    syscallPrintInt();
                    break;

                case SC_ReadFloat:
                    syscallReadFloat();
                    break;

                case SC_PrintFloat:
                    syscallPrintFloat();
                    break;

                case SC_ReadChar:
                    syscallReadChar();
                    break;

                case SC_PrintChar:
                    syscallPrintChar();
                    break;
                
                case SC_ReadString:
                    syscallReadString();
                    break;

                case SC_PrintString:
                    syscallPrintString();
                    break;
                
                case SC_Create:
                    syscallCreate();
                    break;

                case SC_Open:
                    syscallOpen();
                    break;

                case SC_Close:
                    syscallClose();
                    break;

                case SC_Read:
                    syscallRead();
                    break;

                case SC_Write:
                    syscallWrite();
                    break;

                case SC_Delete:
                    syscallDelete();
                    break;

                case SC_Float_To_Buffer:
                    syscallFloat_To_Buffer();
                    break;
                case SC_FloatCompare:
                    syscallFloatCompare();
                    break;
                case SC_ExecProc:
                    ExceptionHandler_Exec();
                    
                    break;
                case SC_JoinProc:
                    ExceptionHandler_Join();
                    
                    break;
                case SC_ExitProc:
                    ExceptionHandler_Exit();
                    break;
                case SC_CreateSemaphore:
                    ExceptionHandler_CreateSemaphore();
                    break;
                case SC_Down:
                    ExceptionHandler_Down();        
                    break;
                case SC_Up:
                    ExceptionHandler_Up(); 
                    break;
            }
            AdvancePC();
            break;
        }

    }

    return;

}
