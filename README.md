// declare a 1 Kb memory with 16 bit width 
// so depth will be 2**6
// MEMORY IS SYNCHRONOUS CKT(sequential)
// behivioral style of coding 
module memory( clk_i, rst_i, wr_rd_i, addr_i, wdata_i, rdata_o, valid_i, ready_o);
  
  parameter WIDTH=16;
  parameter DEPTH=64;
  parameter ADDR_SIZE=$clog2(DEPTH);
  
  input clk_i, rst_i;
  input[ADDR_SIZE:0] addr_i;
  input [WIDTH-1:0]wdata_i;
  output reg [WIDTH-1:0]rdata_o;
  input wr_rd_i;
  input valid_i;
  output reg ready_o;
  reg [WIDTH-1:0]mem[DEPTH-1:0];// array of vectors 

  always@(posedge clk_i) begin
    if(rst_i) begin
      rdata_o<=0;
      ready_o<=0;
      // to make mem =0 it is an array and also one of the reg variable (for loop)
      for( int i=0; i<DEPTH; i++) 
        mem[i]=0;
    end 

      if( valid_i==1) begin 
        ready_o=1;// bcz these will ensure succesfull  wr/rd transaction
        if (wr_rd_i) // to store writedata to paticular memory address 
          mem[addr_i]=wdata_i;
       else// read data from a paticular memory address 
         rdata_o=mem[addr_i];
      end 
        else begin
          ready_o=0;
        end 
  end 
      endmodule 






// TB FOR MEMORY

module tb;
  parameter WIDTH=16;
  parameter DEPTH=64;
  parameter ADDR_SIZE=$clog2(DEPTH);
  
  reg clk_i, rst_i;
  reg[ADDR_SIZE:0] addr_i;
  reg [WIDTH-1:0]wdata_i;
  wire [WIDTH-1:0]rdata_o;
  reg wr_rd_i;
  reg valid_i;
  wire ready_o;
  integer i;
  
  memory #(WIDTH,DEPTH,ADDR_SIZE) dut( clk_i, rst_i, wr_rd_i, addr_i, wdata_i, rdata_o, valid_i, ready_o);// parameter overiding from TB
  initial 
    begin
      clk_i=0;
      forever #5 clk_i=~clk_i;
    end 
  
  task reset_mem();
    begin
      rst_i=1; 
      // TB will drive design inputs to 0 and DUT WILL DRIVE design outputs to 0
      // this ensures that all design inputs and outputs  are 0 when reset gets applied
      addr_i= 0;
      wdata_i =0; 
      wr_rd_i=0;
      valid_i =0;
      @(posedge clk_i);
       rst_i=0;
    end 
  endtask 
  
  // Now i want to write at all location of memory 
  // this can only be done using for loop
  
  task write_mem();
    begin
      for(i=0;i<DEPTH;i++) begin
        @(posedge clk_i)// bcz write/read should happen at posedge of clk 
        addr_i=i;// indexing  where we want to write 
        wdata_i=$random;// now writing random data
        wr_rd_i=1;// write to memeory can only be perfomed when "wr_rd_i=1"
        valid_i=1;// to ensure valid transaction to memory bcz via (valid==1) memory will get to know wether it is a valid transcation
        wait (ready_o==1);// we will wait for memeory to send ready in response for valid 
      end 
      // after accessing all 64 loc and writing into them  i dont want to access any further location so i will make sure by making valid==0 
      @(posedge clk_i);
      valid_i=0;//(L1) as most imp line 
    end
  endtask
  
  task read_mem();
    begin
      for( i=0; i<DEPTH; i=i+1) begin
        @(posedge clk_i);
        addr_i=i;
        wr_rd_i=0;// to enable read 
        valid_i=1;
        wait(ready_o==1);
      end
       @(posedge clk_i);
        valid_i=0;
    end 
  endtask
  
  initial
    begin
      reset_mem();// untill called task reset_mem() won't run
      write_mem();
      read_mem();
    end 
  
  initial begin
  $dumpfile("dump.vcd");
  $dumpvars;
  #10000
  $finish;
end

  

endmodule 


// TO write to paticular locations or to read from a paticular location
module memory( clk_i, rst_i, wr_rd_i, addr_i, wdata_i, rdata_o, valid_i, ready_o);
  
  parameter WIDTH=16;
  parameter DEPTH=64;
  parameter ADDR_SIZE=$clog2(DEPTH);
  
  input clk_i, rst_i;
  input[ADDR_SIZE:0] addr_i;
  input [WIDTH-1:0]wdata_i;
  output reg [WIDTH-1:0]rdata_o;
  input wr_rd_i;
  input valid_i;
  output reg ready_o;
  reg [WIDTH-1:0]mem[DEPTH-1:0];
  integer i;

  always@(posedge clk_i) begin
    if(rst_i) begin
      rdata_o<=0;
      ready_o<=0;
  for(  i=0; i<DEPTH; i++) 
        mem[i]=0;
    end 

      if( valid_i==1) begin 
        ready_o=1;
        if (wr_rd_i) 
          mem[addr_i]=wdata_i;
       else
         rdata_o=mem[addr_i];
      end 
        
        else begin
          ready_o=0;
        end 
  end 
      endmodule 

module tb;
  parameter WIDTH=16;
  parameter DEPTH=64;
  parameter ADDR_SIZE=$clog2(DEPTH);
  
  reg clk_i, rst_i;
  reg[ADDR_SIZE:0] addr_i;
  reg [WIDTH-1:0]wdata_i;
  wire [WIDTH-1:0]rdata_o;
  reg wr_rd_i;
  reg valid_i;
  wire ready_o;
  integer i;
  
  memory #(WIDTH,DEPTH,ADDR_SIZE) dut( clk_i, rst_i, wr_rd_i, addr_i, wdata_i, rdata_o, valid_i, ready_o);// parameter overiding from TB
  initial 
    begin
      clk_i=0;
      forever #5 clk_i=~clk_i;
    end 
  
  task reset_mem();
    begin
      rst_i=1; 
     
      addr_i= 0;
      wdata_i =0; 
      wr_rd_i=0;
      valid_i =0;
      @(posedge clk_i);
       rst_i=0;
    end 
  endtask 
  
 // This code won't run as it is for all locations  
  //for working on specific locations we will provide input argumemts to task 
  task write_mem(input[ADDR_SIZE-1:0]start_location, input[ADDR_SIZE-1:0]num_location);// num_location means number of location i want tom wr/rd 
    begin
      for(i=start_location;i<start_location+num_location;i++) begin // why "start_location+num_location" bcz if i want to write from 10 t0 19 10 will become my start location and if i keep my num_location as 9 for loop wonn't run instead it should be total of both 
        @(posedge clk_i)
        addr_i=i;
        wdata_i=$random;
        wr_rd_i=1;
        valid_i=1;
        wait (ready_o==1);
      end 
      
      @(posedge clk_i);
      valid_i=0;
    end
  endtask
  
  task read_mem(input[ADDR_SIZE-1:0]start_location, input[ADDR_SIZE--1:0]num_location);
    begin
      for( i=start_location; i<start_location+num_location; i=i+1) begin
        @(posedge clk_i);
        addr_i=i;
        wr_rd_i=0;
        valid_i=1;
        wait(ready_o==1);
      end
       @(posedge clk_i);
        valid_i=0;
    end 
  endtask
  
  
  initial
    begin
      reset_mem();
      write_mem('h15,1);
      read_mem('h15,1);//here by changing 2 argumemt i can define no. Of location i want to write 
    end 
  
  
  initial begin
  $dumpfile("dump.vcd");
  $dumpvars;
  #10000
  $finish;
end
endmodule 



module tb;
  parameter WIDTH = 16;
  parameter DEPTH = 64;
  parameter ADDR_SIZE = $clog2(DEPTH);

  reg clk_i, rst_i;
  reg [ADDR_SIZE:0] addr_i;
  reg [WIDTH-1:0] wdata_i;
  wire [WIDTH-1:0] rdata_o;
  reg wr_rd_i;
  reg valid_i;
  wire ready_o;
  integer i;

  memory #(WIDTH, DEPTH, ADDR_SIZE) dut (clk_i, rst_i, wr_rd_i, addr_i, wdata_i, rdata_o, valid_i, ready_o);

  initial begin
    clk_i = 0;
    forever #5 clk_i = ~clk_i;
  end

  task reset_mem();
    begin
      rst_i = 1;
      addr_i = 0;
      wdata_i = 0;
      wr_rd_i = 0;
      valid_i = 0;
      @(posedge clk_i);
      rst_i = 0;
    end
  endtask

  task write_mem(input[ADDR_SIZE-1:0]start_location, input[ADDR_SIZE-1:0]num_location);
    begin
    for(i=start_location;i<start_location+num_location;i++) begin 
        @(posedge clk_i);
        addr_i = i;
        wdata_i = $random;
        wr_rd_i = 1;
        valid_i = 1;
        wait (ready_o == 1);
      end
      @(posedge clk_i);
      valid_i = 0;
    end
  endtask

  task read_mem(input[ADDR_SIZE-1:0]start_location, input[ADDR_SIZE-1:0]num_location);
    begin
       for(i=start_location;i<start_location+num_location;i++) begin 
        @(posedge clk_i);
        addr_i = i;
        wr_rd_i = 0;
        valid_i = 1;
        wait (ready_o == 1);
      end
      @(posedge clk_i);
      valid_i = 0;
    end
  endtask

  initial begin
     reset_mem();
     // want to write into first quarter of memory and read from it   
     write_mem(0,DEPTH/4);
     read_mem(0,DEPTH/4);
    // want to write into second quarter of memory and read from it   
     write_mem(DEPTH/4,DEPTH/4);
    read_mem(DEPTH/4,DEPTH/4);
     // want to write into third quarter of memory and read from it
    write_mem(DEPTH/2,DEPTH/4);
    read_mem(DEPTH/2,DEPTH/4);
     // want to write into fourth quarter of memory and read from it  
    write_mem(3*DEPTH/4,DEPTH/4);
     read_mem(3*DEPTH/4,DEPTH/4);
  end

 initial begin
  $dumpfile("dump.vcd");
  $dumpvars;
  #10000
  $finish;
end
  
endmodule 

// 
  task read_mem(input[ADDR_SIZE-1:0]start_location, input[ADDR_SIZE-1:0]num_location);
    begin
       for(i=start_location;i<start_location+num_location;i++) begin 
        @(posedge clk_i);
        addr_i = i;
        wr_rd_i = 0;
        valid_i = 1;
        wait (ready_o == 1);
      end
      @(posedge clk_i);
      valid_i = 0;
    end
  endtask
// when my memory is divided into 8 equal blocks of size depth/8
  initial begin
     reset_mem();
     // want to write into fifth loc  of memory and read from it   
    write_mem(5*DEPTH/8,DEPTH/8);
    read_mem(5*DEPTH/8,DEPTH/8);
   end


 initial begin
  $dumpfile("dump.vcd");
  $dumpvars;
  #10000
  $finish;
end

endmodule 
