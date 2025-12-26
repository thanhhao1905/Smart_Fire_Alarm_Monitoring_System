````verilog
module uart_mq2_receiver #(
    parameter CLK_FREQ = 40_000_000,
    parameter BAUD_RATE = 9600
)(
    input wire clk,
    input wire rst_n,
    input wire rx,
    
    output reg [3:0] d3,
    output reg [3:0] d2,
    output reg [3:0] d1,
    output reg [3:0] d0,
    output reg temp,
    output reg hum,
    output reg smoke,
    output reg esp32_warning
);

    localparam CLKS_PER_BIT = CLK_FREQ / BAUD_RATE;
    
    // FSM states for UART RX
    localparam IDLE         = 3'd0;
    localparam START_BIT    = 3'd1;
    localparam DATA_BITS    = 3'd2;
    localparam STOP_BIT     = 3'd3;
    localparam CLEANUP      = 3'd4;
    
    reg [2:0] state = IDLE;
    reg [15:0] clk_count = 0;
    reg [2:0] bit_index = 0;
    reg [7:0] rx_byte = 0;
    reg rx_done = 0;
    
    // Synchronize RX signal
    reg rx_data_sync1 = 1'b1;
    reg rx_data_sync2 = 1'b1;
    
    always @(posedge clk) begin
        rx_data_sync1 <= rx;
        rx_data_sync2 <= rx_data_sync1;
    end
    
    // UART byte receiver FSM
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            state <= IDLE;
            clk_count <= 0;
            bit_index <= 0;
            rx_byte <= 0;
            rx_done <= 0;
        end else begin
            rx_done <= 0;
            
            case (state)
                IDLE: begin
                    clk_count <= 0;
                    bit_index <= 0;
                    
                    if (rx_data_sync2 == 1'b0) begin
                        state <= START_BIT;
                    end
                end
                
                START_BIT: begin
                    if (clk_count == (CLKS_PER_BIT - 1) / 2) begin
                        if (rx_data_sync2 == 1'b0) begin
                            clk_count <= 0;
                            state <= DATA_BITS;
                        end else begin
                            state <= IDLE;
                        end
                    end else begin
                        clk_count <= clk_count + 1;
                    end
                end
                
                DATA_BITS: begin
                    if (clk_count < CLKS_PER_BIT - 1) begin
                        clk_count <= clk_count + 1;
                    end else begin
                        clk_count <= 0;
                        rx_byte[bit_index] <= rx_data_sync2;
                        
                        if (bit_index < 7) begin
                            bit_index <= bit_index + 1;
                        end else begin
                            bit_index <= 0;
                            state <= STOP_BIT;
                        end
                    end
                end
                
                STOP_BIT: begin
                    if (clk_count < CLKS_PER_BIT - 1) begin
                        clk_count <= clk_count + 1;
                    end else begin
                        clk_count <= 0;
                        rx_done <= 1;
                        state <= CLEANUP;
                    end
                end
                
                CLEANUP: begin
                    state <= IDLE;
                end
                
                default: begin
                    state <= IDLE;
                end
            endcase
        end
    end
    
    // 3-byte receiver FSM
    localparam WAIT_BYTE1 = 2'd0;
    localparam WAIT_BYTE2 = 2'd1;
    localparam WAIT_BYTE3 = 2'd2;
    localparam PROCESS    = 2'd3;
    
    reg [1:0] rx_state = WAIT_BYTE1;
    reg [7:0] byte1 = 0;
    reg [7:0] byte2 = 0;
    reg [7:0] byte3 = 0;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            rx_state <= WAIT_BYTE1;
            byte1 <= 0;
            byte2 <= 0;
            byte3 <= 0;
            d3 <= 0;
            d2 <= 0;
            d1 <= 0;
            d0 <= 0;
            temp <= 0;
            hum <= 0;
            smoke <= 0;
            esp32_warning <= 0;
        end else begin
            case (rx_state)
                WAIT_BYTE1: begin
                    if (rx_done) begin
                        byte1 <= rx_byte;
                        rx_state <= WAIT_BYTE2;
                    end
                end
                
                WAIT_BYTE2: begin
                    if (rx_done) begin
                        byte2 <= rx_byte;
                        rx_state <= WAIT_BYTE3;
                    end
                end
                
                WAIT_BYTE3: begin
                    if (rx_done) begin
                        byte3 <= rx_byte;
                        rx_state <= PROCESS;
                    end
                end
                
                PROCESS: begin
                    // Extract nibbles from bytes
                    d3 <= byte1[7:4];
                    d2 <= byte1[3:0];
                    d1 <= byte2[7:4];
                    d0 <= byte2[3:0];
                    
                    // Extract control bits
                    temp <= byte3[0];
                    hum <= byte3[1];
                    smoke <= byte3[2];
                    esp32_warning <= byte3[3];
                    
                    rx_state <= WAIT_BYTE1;
                end
                
                default: begin
                    rx_state <= WAIT_BYTE1;
                end
            endcase
        end
    end
endmodule
