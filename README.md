/* Control word specs

bit 0 -------------EN_ALU
bit 1 -------------EN_Mem
bit 2 -------------C0
bits 3 to 7 -------FS 
bit 8 -------------MemWrite
bit 9 -------------RegWrite
bits 10 to 14 -----DA
bits 15 to 19 -----SB
bits 20 to 24 -----SA

RAM address comes from ALU_output

*/

/* Important Note:
   
	Data from the data bus cannot be loaded immediately as
	to the register when being change. This is due to delay.
	Wait a few seconds
*/

module Datapath_Testbench();
// Ports to and from datapath
	reg [25:0] ControlWord;
	reg [63:0] constant;
	wire [63:0] data;
	reg clock, reset;
	wire [3:0] status;
	
	
// Other interconnecting wires
   wire [4:0] SA, SB, DA;
	wire MemWrite, RegWrite;
	wire [63:0] RegAbus, RegBbus;
	wire [63:0] MEM_output;
	wire [4:0] FS;
	wire C0;
	wire [63:0] ALU_output;
	wire EN_ALU, EN_Mem;
	
// Registers and memory being used
	wire [63:0] R0, R1, R2;
   wire [63:0] M0, M1, M2;
	
   assign R2 = dut.regfile.R02;
	assign R1 = dut.regfile.R01;
	assign R0 = dut.regfile.R00;
   assign M0 = dut.data_mem.mem[0];
	assign M1 = dut.data_mem.mem[1];
	assign M2 = dut.data_mem.mem[2];
	
// Instantiation
	DatapathLEGv8 dut (ControlWord, status, constant, data, clock, reset);
	
// Beginning of Testbench; Sets everything to zero
	initial begin
		ControlWord = 26'b0;
		constant = 64'b0;
		clock = 1'b0;
		reset = 1'b1;	
		
	// "Reset" sets all of the registers to zero
		#10 reset = 1'b0; 

	// MEM[0] = 0
	   #11 ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00000,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b1,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b0,/*EN_ALU*/1'b0, /*Bsel*/ 1'b0};
	// MEM[1] = 1
		#11 ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00000,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b0,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};
			constant = 64'd1;
		#1  ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00000,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b1,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b0,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};
							  
	// MEM[2] = 2
		#11 ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00000,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b0,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};
			 constant = 64'd2;
		#1  ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00000,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b1,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b0,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};

	// MEM[2] to REG[2]
		#20 ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00010,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000, /*C0*/1'b0,/*EN_Mem*/1'b1,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};
								
		#1 ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00010,/*RegWrite*/ 1'b1,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000, /*C0*/1'b0,/*EN_Mem*/1'b1,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};
								
		#11 ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00010,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000, /*C0*/1'b0,/*EN_Mem*/1'b1,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};

   // MEM[1] to REG[1]
      #5 constant = 64'd1;
   	#11  	ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00001,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b1,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};
				
		#1    ControlWord = {/*SA*/5'b11111, /*SB*/5'b11111,/*DA*/5'b00001,/*RegWrite*/ 1'b1,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b1,/*EN_ALU*/1'b0, /*Bsel*/ 1'b1};
							
	// REG[0] -> REG[1] + REG[2] (2+1)
		#11  ControlWord = {/*SA*/5'b00010, /*SB*/5'b00001,/*DA*/5'b00000,/*RegWrite*/ 1'b0,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b0,/*EN_ALU*/1'b1, /*Bsel*/ 1'b0};
		#3   ControlWord = {/*SA*/5'b00010, /*SB*/5'b00001,/*DA*/5'b00000,/*RegWrite*/ 1'b1,/*MemWrite*/ 1'b0,
		                  /*FS*/ 5'b01000,/*C0*/1'b0,/*EN_Mem*/1'b0,/*EN_ALU*/1'b1, /*Bsel*/ 1'b0};
		
		#20 ControlWord = 26'b0;
		
	end

		
// Ends the testbench
	initial begin
		#2000 $stop;
	end
	
// Sets clock frequency
	always begin             
		#5 clock = ~clock; 
	end

   assign SA = dut.SA;
	assign SB = dut.SB;
	assign DA = dut.DA;
	assign RegWrite = dut.RegWrite;
	assign MemWrite = dut.MemWrite;
	assign RegAbus = dut.RegAbus;
	assign RegBbus = dut.RegBbus;
	assign FS = dut.FS;
	assign C0 = dut.C0;
	assign ALU_output = dut.ALU_output;
	assign MEM_output = dut.MEM_output;
	assign EN_ALU = dut.EN_ALU;
	assign EN_Mem = dut.EN_Mem;
//	assign data = dut.data;

endmodule
	
