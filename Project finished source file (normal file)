`timescale 1ns / 1ps


module BallBeamTop(
    input  wire CLK100MHZ,
    input  wire [15:0] SW,
    input  wire Echo,
    output wire Trig,
    output wire PWM_Pulse,
    output wire DP,
    output wire G,
    output wire F,
    output wire E,
    output wire D,
    output wire C,
    output wire B,
    output wire A,
    output wire [7:0] Enable
);

    wire [63:0] Result;               // Distance in inches
    wire [18:0] ControllerCompare;

    // Automatic distance measurer
    Measurer meas(
        .CLK100MHZ(CLK100MHZ),
        .Trig(Trig),
        .Echo(Echo),
        .DP(DP),
        .G(G),
        .F(F),
        .E(E),
        .D(D),
        .C(C),
        .B(B),
        .A(A),
        .Enable(Enable),
        .Result(Result)
    );

    // PID/Proportional controller
    Controller ctrl(
        .CLK(CLK100MHZ),
        .AutoMode(~SW[15]),
        .ManualPosition({SW,4'b0000}),
        .SetPointInches(SW[7:0]),
        .BallInches(Result[15:0]),
        .CompareOut(ControllerCompare)
    );

    // PWM output to servo motor
    PWM pwm_inst(
        .CLK100MHZ(CLK100MHZ),
        .SW(ControllerCompare),
        .PWM_Pulse(PWM_Pulse)
    );

endmodule



module PWM(
    input CLK100MHZ,
    input [18:0] SW,
    output reg PWM_Pulse
);
    wire [31:0] Compare_Value;

    PWM_Format format_inst({13'b0,SW}, Compare_Value);             

    reg [31:0] Count = 0;

    always @(posedge CLK100MHZ) begin
        Count <= Count + 1;
        if (Count == 32'd500000) Count <= 0; // 20 ms period
    end

    always @(*) begin
        if(Compare_Value > Count)
            PWM_Pulse = 1;
        else
            PWM_Pulse = 0;
    end
endmodule



module PWM_Format(
    input [31:0] SW,
    output reg [31:0] Compare_Value
);
    always @(*) begin
        if(SW < 32'd100000)
            Compare_Value = 32'd100000;
        else if(SW > 32'd200000)
            Compare_Value = 32'd200000;
        else
            Compare_Value = SW;
    end
endmodule



module Measurer(
    input CLK100MHZ,
    input Echo,
    output reg Trig,
    output wire DP,G,F,E,D,C,B,A,
    output wire [7:0] Enable,
    output wire [63:0] Result
);
    reg [31:0] Count = 0;
    reg [31:0] Distance_Counter = 0;
    reg [63:0] Distance = 0;
    wire [15:0] BCD;

   
    parameter PERIOD = 32'd5_000_000;

    always @(posedge CLK100MHZ) begin
        Count <= Count + 1;

        if (Echo)
            Distance_Counter <= Distance_Counter + 1;

        if (Count >= PERIOD) begin
            Distance <= Distance_Counter;
            Distance_Counter <= 0;
            Count <= 0;
        end
    end

  
    always @(*) begin
        if (Count < 1000)
            Trig = 1;
        else
            Trig = 0;
    end


    division div_inst(Distance, 64'd148, Result);
    bin2bcd bcd_inst(Result[13:0], BCD[15:0]);
    SSEG disp_inst(DP,G,F,E,D,C,B,A,Enable,CLK100MHZ,BCD);
endmodule



module division(
    input wire [63:0] A,
    input wire [63:0] B,
    output wire [63:0] Res2
);
    reg [63:0] Res = 0;
    reg [63:0] a1, b1;
    reg [64:0] p1;
    integer i;

    always @ (A or B) begin
        a1 = A;
        b1 = B;
        p1 = 0;
        for (i = 0; i < 64; i = i + 1) begin
            p1 = {p1[63:0], a1[63]};
            a1[63:1] = a1[62:0];
            p1 = p1 - b1;
            if (p1[64] == 1) begin
                a1[0] = 0;
                p1 = p1 + b1;
            end else begin
                a1[0] = 1;
            end
        end
        Res = a1;
    end
    assign Res2 = Res;
endmodule



module bin2bcd(
    input [13:0] bin,
    output reg [15:0] bcd
);
    integer i;
    always @(bin) begin
        bcd = 0;
        for (i=0; i<14; i=i+1) begin
            if (bcd[3:0]   >= 5) bcd[3:0]   = bcd[3:0]   + 3;
            if (bcd[7:4]   >= 5) bcd[7:4]   = bcd[7:4]   + 3;
            if (bcd[11:8]  >= 5) bcd[11:8]  = bcd[11:8]  + 3;
            if (bcd[15:12] >= 5) bcd[15:12] = bcd[15:12] + 3;
            bcd = {bcd[14:0], bin[13-i]};
        end
    end
endmodule



module SSEG(
    output reg DP,G,F,E,D,C,B,A,
    output reg [7:0] Enable,
    input wire CLK100MHZ,
    input wire [15:0] BCD
);
    reg [3:0] Location = 0;
    reg [3:0] Digit;
    wire Clk_Multi;

    CLK100MHZ_divider clkdiv(.CLK100MHZ(CLK100MHZ), .New_Clock(Clk_Multi));

    always @(*) begin
        case(Digit)
            4'h0: {G,F,E,D,C,B,A} = 7'b1000000;
            4'h1: {G,F,E,D,C,B,A} = 7'b1001111;
            4'h2: {G,F,E,D,C,B,A} = 7'b0100100;
            4'h3: {G,F,E,D,C,B,A} = 7'b0110000;
            4'h4: {G,F,E,D,C,B,A} = 7'b0011001;
            4'h5: {G,F,E,D,C,B,A} = 7'b0010010;
            4'h6: {G,F,E,D,C,B,A} = 7'b0000010;
            4'h7: {G,F,E,D,C,B,A} = 7'b1111000;
            4'h8: {G,F,E,D,C,B,A} = 7'b0000000;
            4'h9: {G,F,E,D,C,B,A} = 7'b0011000;
            4'hA: {G,F,E,D,C,B,A} = 7'b0001000;
            4'hB: {G,F,E,D,C,B,A} = 7'b0000011;
            4'hC: {G,F,E,D,C,B,A} = 7'b1000110;
            4'hD: {G,F,E,D,C,B,A} = 7'b0100001;
            4'hE: {G,F,E,D,C,B,A} = 7'b0000110;
            4'hF: {G,F,E,D,C,B,A} = 7'b0001110;
        endcase
    end

    always @(posedge Clk_Multi) begin
        Location <= Location + 1;
        case(Location)
            0: begin Enable <= 8'b11111110; {DP,Digit} <= {1'b1,BCD[3:0]}; end
            1: begin Enable <= 8'b11111101; {DP,Digit} <= {1'b1,BCD[7:4]}; end
            2: begin Enable <= 8'b11111011; {DP,Digit} <= {1'b0,BCD[11:8]}; end
            3: begin Enable <= 8'b11110111; {DP,Digit} <= {1'b1,BCD[15:12]}; end
            default: begin Enable <= 8'b11111111; end
        endcase
    end
endmodule



module CLK100MHZ_divider(
    input wire CLK100MHZ,
    output reg New_Clock
);
    reg [31:0] count = 0;
    always @(posedge CLK100MHZ) begin
        count <= count + 1;
        if (count == 31'd10000) begin
            New_Clock <= ~New_Clock;
            count <= 0;
        end
    end
endmodule



module Controller(
    input wire CLK,
    input wire AutoMode,
    input wire [19:0] ManualPosition,
    input wire [7:0] SetPointInches,
    input wire [15:0] BallInches,
    output reg [18:0] CompareOut
);
    localparam integer CENTER = 150000;
    localparam integer MINC = 100000;
    localparam integer MAXC = 200000;
    localparam integer KP = 10;

    reg signed [31:0] error;
    reg signed [31:0] control;

    always @(posedge CLK) begin
        if (AutoMode == 1'b0) begin
            error <= $signed({8'b0,SetPointInches}) - $signed(BallInches);
            control <= CENTER + error * KP;
            if (control < MINC) control <= MINC;
            if (control > MAXC) control <= MAXC;
            CompareOut <= control[18:0];
        end else begin
            CompareOut <= ManualPosition[18:0];
        end
    end
endmodule
