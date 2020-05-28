module
newatm1(wen,ren,ip,locn,clk,rst,userpswd,useractno,useramt,invalid,invalidpswd,nobalance,lang,ch,tr);
 input clk,rst,ch,tr;
 input [15:0] useractno;
 input [7:0] userpswd;
 input [15:0] useramt;
 input [1:0] lang;
 input [39:0] ip;
 input ren,wen;
 input [3:0] locn;

 output invalid,invalidpswd,nobalance;
 reg invalid,invalidpswd,nobalance,flag;
 reg [3:0] addr,addr1;
 reg [2:0] state;
 reg [39:0] mem [9:0];
 reg [15:0] balance,acctno;
 reg [7:0] pswd;
 always @(posedge clk )
 begin
 if(wen)
 begin
 mem[locn]<=ip;
 end
 if(ren )
 begin
 acctno=mem[addr][39:24];
 pswd=mem[addr][23:16]; 
 balance=mem[addr][15:0];
 end
 end
 always@(posedge clk)
 begin
 if(rst)
 begin
 addr<=4'b0000;
 invalid<=0;
 flag<=0;
 invalidpswd<=0;
 nobalance<=0;
 state<=3'b000;
 end
 else
 begin
 case(state)

 3'b000:
 begin

 //$display("Welcome to state bank of India");
 //$display("Enter the account no:");
 if(useractno==acctno)
 begin

 state=3'b001;
 flag=1; 
 addr1=addr;
 end
 else if(addr==4'd10)
 begin
 state=3'b111;
 invalid=1;
 end
 else
 addr=addr+1;
 end
 3'b001:
 begin
 //$display("Enter the password");
 if(userpswd==pswd)
 state=3'b010;
 else
 begin
 //$display("Your password is wrong");
 invalidpswd<=1;
 end

 end
 3'b010:
 begin
 //$display("Enter the language 1:English 2:Malayalam 3:Hindi");
 if(lang==2'b00)
 state=3'b011;
 end 
 3'b011:
 begin
 //$display("Enter the transaction required 0:Balance Enquiry 1:withdrawal");
 if(ch==0)
 state=3'b100;
 else
 state=3'b101;
 end
 3'b100:
 begin
 //$display("The balance is %h",mem[addr1][15:0]);
 state=3'b110;
 end
 3'b101:
 begin
 if(useramt>40'h4000)
 begin
 nobalance<=1;
 //$display("Invalid amount");
 end
 else if((balance-useramt)<10'h500)
 begin
 //$display("No balance");
 end

 else
 begin
 
 mem[addr1]<={acctno,pswd,mem[addr1][15:0]-useramt};


 end
 state=3'b110;
 end
 3'b110:
 begin
 //$display("Do you want another transaction");
 if(tr==1)
 state=3'b011;


 end
 3'b111:
 begin
 //$display("Invalid account");
 state=3'b000;
 end
 endcase

 end

 end


 always@(state)
 begin 
 case(state)

 3'b000:
 begin
 $display("Welcome to state bank of India");
 $display("Enter the account no:");

 end
 3'b001:
 begin
 $display("%h",mem[addr1][39:24]);
 $display("Enter the password");
 $display("%h",pswd);
 if(userpswd!=mem[addr1][23:16])
 $display("INvalid password");
 end
 3'b010:
 begin
 $display("Enter the language 1:English 2:Malayalam 3:Hindi");
 end
 3'b011:
 begin

 $display("Enter the transaction required 0:Balance Enquiry 1:withdrawal");
 $monitor("%b",ch);
 end
 3'b100:
 begin 
 $display("The balance is %h",mem[addr1][15:0]);
 end
 3'b101:
 begin
 $display("Enter the amount");
 $display("%h",useramt);
 if(useramt>40'h4000)
 begin
 nobalance<=1;
 $display("Invalid amount");
 end
 else if((balance-useramt)<10'h500)
 begin
 $display("No balance");
 end
 end
 3'b110:
 begin
 $display("Do you want another transaction");
 $monitor("%b",tr);
 end
 3'b111:
 begin
 $display("Invalid account");
 end
 endcase
 end
endmodule 
