# DIGITAL-FILTER-DESIGN-
#fixed point FIR design 
module FIR_Filter (
    input wire clk,
    input wire reset,
    input wire signed [15:0] data_in,
    output reg signed [15:0] data_out
);

// Filter coefficients (Q1.15 format - 1 sign bit, 0 integer bits, 15 fractional bits)
parameter signed [15:0] coeff [0:7] = '{
    16'h0A3D, 16'h1A9B, 16'h25ED, 16'h2B5F,
    16'h2B5F, 16'h25ED, 16'h1A9B, 16'h0A3D
};

// Delay line (shift register)
reg signed [15:0] delay_line [0:7];
integer i;

// Multiply-accumulate
reg signed [31:0] accumulator;

always @(posedge clk or posedge reset) begin
    if (reset) begin
        // Reset delay line
        for (i = 0; i < 8; i = i + 1)
            delay_line[i] <= 16'b0;
        accumulator <= 32'b0;
        data_out <= 16'b0;
    end
    else begin
        // Shift new data into delay line
        for (i = 7; i > 0; i = i - 1)
            delay_line[i] <= delay_line[i-1];
        delay_line[0] <= data_in;
        
        // Perform multiply-accumulate
        accumulator <= 
            (delay_line[0] * coeff[0]) +
            (delay_line[1] * coeff[1]) +
            (delay_line[2] * coeff[2]) +
            (delay_line[3] * coeff[3]) +
            (delay_line[4] * coeff[4]) +
            (delay_line[5] * coeff[5]) +
            (delay_line[6] * coeff[6]) +
            (delay_line[7] * coeff[7]);
            
        // Output the result (with rounding)
        data_out <= accumulator[30:15] + accumulator[14];
    end
end

endmodule
