````verilog
module lcd_write_cmd_data(
    input       clk_1MHz,
    input       rst_n,
    input [7:0] data,
    input       cmd_data,
    input       ena,
    input [6:0] i2c_addr,
    inout       sda,
    output      scl,
    output      done,
    output      sda_en
);

    localparam  DELAY = 50;
    reg [20:0]  cnt;
    reg         cnt_clr;

    // FSM states
    localparam  WaitEn              = 0,
                Write_Addr          = 1,
                Wait_AddrDone       = 2,
                Write_HighNibble1   = 3,
                Wait_High1Done      = 4,
                Delay_CMD1          = 5,
                Write_HighNibble2   = 6,
                Wait_High2Done      = 7,
                Write_LowNibble1    = 8,
                Wait_Low1Done       = 9,
                Delay_CMD2          = 10,
                Write_LowNibble2    = 11,
                Wait_Low2Done       = 12,
                Done                = 13;
    
    reg [3:0]   state, next_state;
    reg         en_write;
    wire        en_i2cwrite = en_write;
    wire        i2c_done;
    reg [7:0]   i2c_data;

    reg         start_frame;
    reg         stop_frame;

    // Microsecond counter
    always @(posedge clk_1MHz, negedge rst_n) begin
        if (!rst_n)
            cnt <= 21'd0;
        else if (cnt_clr)
            cnt <= 21'd0;
        else
            cnt <= cnt + 1'b1;
    end

    // State machine
    always @(posedge clk_1MHz, negedge rst_n) begin
        if (!rst_n)
            state <= WaitEn;
        else
            state <= next_state;
    end

    // Next state logic
    always @(*) begin
        if (!rst_n)
            next_state = WaitEn;
        else begin
            case (state)
                WaitEn:             next_state = ena ? Write_Addr : WaitEn;
                Write_Addr:         next_state = Wait_AddrDone;
                Wait_AddrDone:      next_state = i2c_done ? Write_HighNibble1 : Wait_AddrDone;
                Write_HighNibble1:  next_state = Wait_High1Done;
                Wait_High1Done:     next_state = i2c_done ? Delay_CMD1 : Wait_High1Done;
                Delay_CMD1:         next_state = (cnt == DELAY) ? Write_HighNibble2 : Delay_CMD1;
                Write_HighNibble2:  next_state = Wait_High2Done;
                Wait_High2Done:     next_state = i2c_done ? Write_LowNibble1 : Wait_High2Done;
                Write_LowNibble1:   next_state = Wait_Low1Done;
                Wait_Low1Done:      next_state = i2c_done ? Delay_CMD2 : Wait_Low1Done;
                Delay_CMD2:         next_state = (cnt == DELAY) ? Write_LowNibble2 : Delay_CMD2;
                Write_LowNibble2:   next_state = Wait_Low2Done;
                Wait_Low2Done:      next_state = i2c_done ? Done : Wait_Low2Done;
                Done:               next_state = WaitEn;
            endcase
        end
    end

    // Output logic
    always @(posedge clk_1MHz, negedge rst_n) begin
        if (!rst_n) begin
            i2c_data <= 8'd0;
            en_write <= 1'b0;
            cnt_clr  <= 1'b1;
        end else begin
            case (state)
                WaitEn: begin
                    i2c_data    <= 8'd0;
                    en_write    <= 1'b0;
                    cnt_clr     <= 1'b1;
                end
                Write_Addr: begin
                    start_frame <= 1'b1;
                    stop_frame  <= 1'b0;
                    i2c_data    <= {i2c_addr, 1'b0};
                    en_write    <= 1'b1;
                end
                Wait_AddrDone: begin
                    en_write    <= 1'b0;
                end
                Write_HighNibble1: begin
                    start_frame <= 1'b0;
                    stop_frame  <= 1'b0;
                    i2c_data    <= (data & 8'hF0) | 8'h0C | cmd_data;
                    en_write    <= 1'b1;
                end
                Wait_High1Done: begin
                    en_write    <= 1'b0;
                end
                Delay_CMD1: begin
                    cnt_clr     <= 1'b0;
                end
                Write_HighNibble2: begin
                    start_frame <= 1'b0;
                    stop_frame  <= 1'b0;
                    i2c_data    <= ((data & 8'hF0) | 8'h0C | cmd_data) & 8'hFB;
                    en_write    <= 1'b1;
                    cnt_clr     <= 1'b1;
                end
                Wait_High2Done: begin
                    en_write    <= 1'b0;
                end
                Write_LowNibble1: begin
                    start_frame <= 1'b0;
                    stop_frame  <= 1'b0;
                    i2c_data    <= ((data & 8'h0F) << 4) | 8'h0C | cmd_data;
                    en_write    <= 1'b1;
                end
                Wait_Low1Done: begin
                    en_write    <= 1'b0;
                end
                Delay_CMD2: begin
                    cnt_clr     <= 1'b0;
                end
                Write_LowNibble2: begin
                    start_frame <= 1'b0;
                    stop_frame  <= 1'b1;
                    i2c_data    <= (((data & 8'h0F) << 4) | 8'h0C | cmd_data) & 8'hFB;
                    en_write    <= 1'b1;
                    cnt_clr     <= 1'b1;
                end
                Wait_Low2Done: begin
                    en_write    <= 1'b0;
                end
            endcase
        end
    end

    assign done = (state == Done);

    i2c_writeframe i2c_writframe_inst(
        .clk_1MHz   (clk_1MHz),
        .rst_n      (rst_n),
        .en_write   (en_i2cwrite),
        .start_frame(start_frame),
        .stop_frame (stop_frame),
        .data       (i2c_data),
        .sda        (sda),
        .scl        (scl),
        .done       (i2c_done),
        .sda_en     (sda_en)
    );
endmodule
