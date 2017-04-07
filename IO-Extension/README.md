# emPC-A/iMX6 IO-Extension Library (8 digital IO, 4 analog In, 4 analog Out)

you can import this library using the export file: "Device.Plc Logic.Application.POU_DIGANALOGIO.export" 

### Usage
```
io( digout:=digout, digin=>digin, error=>err, aout1:=aout[1], aout2:=aout[2], aout3:=aout[3], aout4:=aout[4], ain1=>ain[1], ain2=>ain[2], ain3=>ain[3], ain4=>ain[4] );
```

### Required Libraries
   * CmpErrors
   * SysShm 
   * SysTypes

## Function Block POU_DIGANALOGIO

### Declaration
```
FUNCTION_BLOCK POU_DIGANALOGIO
VAR_INPUT	
	digout : BYTE; // 8bit digital output 
	aout1 : INT; // 16bit (signed) analog output (-10V .. +10V)
	aout2 : INT; // 16bit (signed) analog output (-10V .. +10V)
	aout3 : INT; // 16bit (signed) analog output (-10V .. +10V)
	aout4 : INT; // 16bit (signed) analog output (-10V .. +10V)
END_VAR
VAR_OUTPUT
	digin : BYTE; // 8bit digital input
	ain1 : INT; // 16bit (signed) analog input (-10V .. +10V)
	ain2 : INT; // 16bit (signed) analog input (-10V .. +10V)
	ain3 : INT; // 16bit (signed) analog input (-10V .. +10V)
	ain4 : INT; // 16bit (signed) analog input (-10V .. +10V)
	error : BOOL:=FALSE;
END_VAR
VAR
	ulSize: DWORD;
    result: DWORD; 
	tmp: BYTE;    
	shm : DWORD := 0;
	physaddr : DWORD := 16#0C000000;
	physaddrsize : DWORD := 16;
	
	pVirt : POINTER TO UDINT;
	pVirtDigInput : POINTER TO UDINT;
	pVirtDigOutput : POINTER TO UDINT;
	
	pVirtAInput1 : POINTER TO INT;
	pVirtAInput2 : POINTER TO INT;
	pVirtAInput3 : POINTER TO INT;
	pVirtAInput4 : POINTER TO INT;
	
	pVirtAOutput1 : POINTER TO INT;
	pVirtAOutput2 : POINTER TO INT;
	pVirtAOutput3 : POINTER TO INT;
	pVirtAOutput4 : POINTER TO INT;
	
END_VAR

```


### Implementation
```
IF shm=0 THEN
	ulSize := physaddrsize;
	shm := SysShm.SysSharedMemoryCreate( 'InternalRegs_JanzDebug_PCIBusNr1', physaddr , ADR(ulSize), ADR(result) );
	IF shm<>0 THEN	 
		pVirt:=SysSharedMemoryGetPointer( shm, ADR(result));
		IF (result=CmpErrors.Errors.ERR_OK) THEN
			pVirtDigInput:=pVirt+16#20;
			pVirtDigOutput:=pVirt+16#20;
			
			pVirtAInput1 := pVirt + 16#40;
			pVirtAInput2 := pVirt + 16#42;
			pVirtAInput3 := pVirt + 16#44;
			pVirtAInput4 := pVirt + 16#46;
			
			pVirtAOutput1 := pVirt + 16#70;
			pVirtAOutput2 := pVirt + 16#72;
			pVirtAOutput3 := pVirt + 16#74;
			pVirtAOutput4 := pVirt + 16#66; // + LDAC			
		ELSE
			pVirt:=0;
		END_IF
	END_IF
END_IF

IF shm<>0 AND pVirt<>0 THEN	 
   		
	// read digital input values from register
	digin:=DINT_TO_BYTE(pVirtDigInput^ AND 16#FF);
	
	// read analog inputs
	ain1:=pVirtAInput1^;
	ain2:=pVirtAInput2^;
	ain3:=pVirtAInput3^;
	ain4:=pVirtAInput4^;
 
	// ------------------------------
	
	// write digital output values to register 
	pVirtDigOutput^ := digout;
	
	// write analog outputs
	pVirtAOutput1^ := aout1;
	pVirtAOutput2^ := aout2;
	pVirtAOutput3^ := aout3;
	pVirtAOutput4^ := aout4; // + LDAC = set all 4 outputs at the same time
		 			
ELSE
	error:=TRUE;
END_IF          
    
```
