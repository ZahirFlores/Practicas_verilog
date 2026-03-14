# Portafolio de Prácticas - Verilog - Bryan Zahir Flores Pérez

Este repositorio contiene las prácticas desarrolladas en Verilog, incluyendo el código principal, testbench y capturas del funcionamiento y simulación.

---

## Práctica 1: Números Primos

### Archivos Normales
```verilog
module nump (
    input  [3:0] SW,  // Los 4 switches de entrada
    output [0:0] LEDR // El LED rojo de salida (LEDR[0])
);

    // Lógica para detectar números primos entre 0 y 15:
    // Primos: 2 (0010), 3 (0011), 5 (0101), 7 (0111), 11 (1011), 13 (1101)
    assign LEDR[0] = (SW == 4'd2)  | 
                     (SW == 4'd3)  |
                     (SW == 4'd5)  |
                     (SW == 4'd7)  |
                     (SW == 4'd11) |
                     (SW == 4'd13);
                            
endmodule
```

### Funcionamiento
![Imagen del funcionamiento de la Práctica 1](Numeros_Primos.jfif)

### Código del Testbench
```verilog
module nump_tb; 

    // Entradas (reg) y salidas (wire)
    reg [3:0] SW_tb;
    wire [0:0] LEDR_tb;

    // Aquí conectamos tu módulo original, que se llama "nump"
    nump dut (
        .SW(SW_tb),
        .LEDR(LEDR_tb)
    );

    // Bloque initial con las pruebas paso a paso
    initial begin
        SW_tb = 4'd0;  #10; // 0 - Apagado
        SW_tb = 4'd1;  #10; // 1 - Apagado
        SW_tb = 4'd2;  #10; // 2 - ENCENDIDO
        SW_tb = 4'd3;  #10; // 3 - ENCENDIDO
        SW_tb = 4'd4;  #10; // 4 - Apagado
        SW_tb = 4'd5;  #10; // 5 - ENCENDIDO
        SW_tb = 4'd6;  #10; // 6 - Apagado
        SW_tb = 4'd7;  #10; // 7 - ENCENDIDO
        SW_tb = 4'd8;  #10; // 8 - Apagado
        SW_tb = 4'd11; #10; // 11 - ENCENDIDO
        SW_tb = 4'd13; #10; // 13 - ENCENDIDO
        SW_tb = 4'd15; #10; // 15 - Apagado

        $stop; 
    end

endmodule
```

### Simulación
![Imagen de la simulación de la Práctica 1](nump_t.jfif)

---

## Práctica 2: BCD 4 displays

### Archivos Normales
```verilog
module BCD_4displays #(parameter N_in=10, N_out=7)(
    input  [N_in-1:0] bin_in, // Entrada binaria de 10 bits (0 a 1023)
    output [N_out-1:0] D_un, D_de, D_ce, D_mi
);

    // Cables internos para guardar cada dígito en formato BCD (4 bits cada uno)
    wire [3:0] unidades, decenas, centenas, millares;

    // 1. CONVERSIÓN DE BINARIO A BCD
    assign millares =  bin_in / 1000;
    assign centenas = (bin_in % 1000) / 100;
    assign decenas  = (bin_in % 100) / 10;
    assign unidades =  bin_in % 10;

    // 2. INSTANCIACIÓN DE LOS DECODIFICADORES DE 7 SEGMENTOS
    bcd_to_7seg dec_mi (.bcd(millares), .seg(D_mi));
    bcd_to_7seg dec_ce (.bcd(centenas), .seg(D_ce));
    bcd_to_7seg dec_de (.bcd(decenas),  .seg(D_de));
    bcd_to_7seg dec_un (.bcd(unidades), .seg(D_un));

endmodule


//Decodificador de BCD a 7 Segmentos
module bcd_to_7seg (
    input  [3:0] bcd,
    output reg [6:0] seg
);
    // Lógica para displays 
    always @(*) begin
        case(bcd)
            4'd0: seg = 7'b1000000; // Muestra '0'
            4'd1: seg = 7'b1111001; // Muestra '1'
            4'd2: seg = 7'b0100100; // Muestra '2'
            4'd3: seg = 7'b0110000; // Muestra '3'
            4'd4: seg = 7'b0011001; // Muestra '4'
            4'd5: seg = 7'b0010010; // Muestra '5'
            4'd6: seg = 7'b0000010; // Muestra '6'
            4'd7: seg = 7'b1111000; // Muestra '7'
            4'd8: seg = 7'b0000000; // Muestra '8'
            4'd9: seg = 7'b0010000; // Muestra '9'
            default: seg = 7'b1111111; // Apagado para cualquier otro valor
        endcase
    end
endmodule
```

### Funcionamiento
![Imagen del funcionamiento de la Práctica 2](dp1.jfif)
![Imagen del funcionamiento de la Práctica 2](dp2.jfif)

### Código del Testbench
```verilog
module BCD_4displays_tb;

    //Declaramos 
    parameter N_in = 10;
    parameter N_out = 7;

    //Entradas 
    reg  [N_in-1:0]  tb_bin_in;
    
    wire [N_out-1:0] tb_D_un;
    wire [N_out-1:0] tb_D_de;
    wire [N_out-1:0] tb_D_ce;
    wire [N_out-1:0] tb_D_mi;

    //le pasamos los parámetros explícitamente
    BCD_4displays #(
        .N_in(N_in),
        .N_out(N_out)
    ) dut (
        .bin_in(tb_bin_in),
        .D_un(tb_D_un),
        .D_de(tb_D_de),
        .D_ce(tb_D_ce),
        .D_mi(tb_D_mi)
    );

    //Bloque initial 
    initial begin
        tb_bin_in = 10'd0;    #10; // Prueba de 0
        tb_bin_in = 10'd7;    #10; // Prueba de un dígito (7)
        tb_bin_in = 10'd42;   #10; // Prueba de dos dígitos (42)
        tb_bin_in = 10'd358;  #10; // Prueba de tres dígitos (358)
        tb_bin_in = 10'd999;  #10; // Límite de tres dígitos (999)
        tb_bin_in = 10'd1000; #10; // Salto a millares (1000)
        tb_bin_in = 10'd1023; #10; // Máximo con 10 interruptores (1023)

        $stop; // Detiene la simulación
    end

endmodule
```

### Simulación
![Imagen de la simulación de la Práctica 2](BCD_t.jfif)

---

## Práctica 3: Contador 0 a 100 ascendente y descendente

### Archivos Normales
```verilog
module contador_top2 (
    input        clk,       // PIN_P11 (50MHz)
    input        rst_n,     // PIN_B8  (KEY0)
    input        btn_load,  // PIN_A7  (KEY1)
    input [6:0]  sw,        // SW0-SW6
    input        sw_dir,    // SW7 (o SW8 según tu descripción)
    output [7:0] hex0,      
    output [7:0] hex1,      
    output [7:0] hex2       
);

    // --- DIVISOR DE RELOJ ---
    reg [22:0] count_clk = 0;
    reg pulse_5hz = 0; 

    always @(posedge clk) begin
        if (count_clk >= 23'd4999999) begin 
            count_clk <= 23'd0;
            pulse_5hz <= 1'b1;
        end else begin
            count_clk <= count_clk + 1'b1;
            pulse_5hz <= 1'b0;
        end
    end

    // --- LÓGICA DE CONTEO ---
    reg [7:0] valor = 8'd0; 
    reg dir_actual = 1'b0; // Registro para guardar la dirección cargada

    always @(posedge clk) begin
        // Reset maestro
        if (rst_n == 1'b0) begin
            valor <= 8'd0;
            dir_actual <= 1'b0;
        end
        // Carga de valor Y dirección (Al presionar el botón)
        else if (btn_load == 1'b0) begin  
            dir_actual <= sw_dir; // AQUÍ guardamos la dirección del switch
            if (sw > 7'd100) valor <= 8'd100;
            else valor <= {1'b0, sw};
        end 
        // Conteo automático
        else if (pulse_5hz) begin
            // Usamos 'dir_actual' en lugar de 'sw_dir' directamente
            if (dir_actual == 1'b0) begin : ARRIBA
                if (valor >= 8'd100) valor <= 8'd0;
                else valor <= valor + 1'b1;
            end 
            else begin : ABAJO
                if (valor == 8'd0 || valor > 8'd100) valor <= 8'd100;
                else valor <= valor - 1'b1;
            end
        end
    end

    // --- SEPARACIÓN EN DÍGITOS ---
    wire [3:0] unidades = valor % 10;
    wire [3:0] decenas  = (valor / 10) % 10;
    wire [3:0] centenas = (valor / 100);

    seven_seg_decoder d0 (unidades, hex0);
    seven_seg_decoder d1 (decenas, hex1);
    seven_seg_decoder d2 (centenas, hex2);

endmodule

module seven_seg_decoder (
    input [3:0] bcd,
    output reg [7:0] seg
);
    always @(*) begin
        case (bcd)
            4'h0: seg = 8'b1100_0000;
            4'h1: seg = 8'b1111_1001;
            4'h2: seg = 8'b1010_0100;
            4'h3: seg = 8'b1011_0000;
            4'h4: seg = 8'b1001_1001;
            4'h5: seg = 8'b1001_0010;
            4'h6: seg = 8'b1000_0010;
            4'h7: seg = 8'b1111_1000;
            4'h8: seg = 8'b1000_0000;
            4'h9: seg = 8'b1001_0000;
            default: seg = 8'b1111_1111;
        endcase
    end
endmodule
```

### Funcionamiento
![Imagen del funcionamiento de la Práctica 3](cont1.jfif)
![Imagen del funcionamiento de la Práctica 3](cont2.jfif)

### Código del Testbench
```verilog
module contador_top2_tb();

    // Señales de estímulo
    reg clk;
    reg rst_n;
    reg btn_load;
    reg [6:0] sw;
    reg sw_dir;

    // Señales de salida
    wire [7:0] hex0, hex1, hex2;

    // Instancia del módulo a probar 
    contador_top2 DUT (
        .clk(clk),
        .rst_n(rst_n),
        .btn_load(btn_load),
        .sw(sw),
        .sw_dir(sw_dir),
        .hex0(hex0),
        .hex1(hex1),
        .hex2(hex2)
    );

    // Generador de Reloj de 50MHz (Periodo de 20ns)
    always #10 clk = ~clk;

    initial begin
        // Inicialización
        clk = 0;
        rst_n = 0;      // Empezamos en reset
        btn_load = 1;   // Botones en la DE10-Lite son lógica negativa (1 = suelto)
        sw = 7'd0;
        sw_dir = 0;     // Dirección inicial: Arriba

        // Esperar un poco y soltar reset
        #100 rst_n = 1;
        
        // Cargar el número 15 para contar hacia ARRIBA
        #50;
        sw = 7'd15;
        sw_dir = 0;     // Queremos que suba
        #20;
        btn_load = 0;   // Presionamos LOAD
        #100;           // Mantener presionado un momento
        btn_load = 1;   // Soltamos LOAD
        
        // Cambiar el Switch de dirección SIN darle a LOAD 
        // Aquí probamos que NO cambie el sentido del conteo todavía
        #1000;
        sw_dir = 1;     // Cambiamos el switch físicamente a ABAJO
        #500;           // Esperamos... el contador debería seguir subiendo (16, 17...)
        
        // Aplicar el cambio de dirección con LOAD
        sw = 7'd50;     // Cambiamos a 50
        btn_load = 0;   // Presionamos LOAD (ahora sí debe tomar el sw_dir = 1)
        #100;
        btn_load = 1;

        // Finalizar simulación
        #2000;
        $stop;          // Detiene la simulación en ModelSim/Questa
    end

    // Monitor para ver los valores en la consola de simulación
    initial begin
        $monitor("Tiempo: %t | Valor: %d | Dir_Actual: %b | Hex: %h %h %h", 
                 $time, uut.valor, uut.dir_actual, hex2, hex1, hex0);
    end

endmodule
```

### Simulación
![Imagen de la simulación de la Práctica 3](contador_t.jfif)

---

## Práctica 4: Password

### Archivos Normales
```verilog
module password (
    input wire clk,          // KEY[0] 
    input wire rst_n,        // KEY[1] 
    input wire [3:0] sw,     // SW[3:0] 
    output reg [6:0] hex3,   // Display HEX3
    output reg [6:0] hex2,   // Display HEX2
    output reg [6:0] hex1,   // Display HEX1
    output reg [6:0] hex0    // Display HEX0
);

    //Configuración
    parameter DIGITO1 = 4'd5;
    parameter DIGITO2 = 4'd4;
    parameter DIGITO3 = 4'd0;
    parameter DIGITO4 = 4'd0;

    // Definición de los estados
    parameter S_INIT = 3'd0; 
    parameter S_N1   = 3'd1; 
    parameter S_N2   = 3'd2; 
    parameter S_N3   = 3'd3; 
    parameter S_GOOD = 3'd4; 
    parameter S_BAD  = 3'd5; 

    reg [2:0] state, next_state;
    
    function [6:0] sseg(input [3:0] bcd);
        case (bcd)
            4'h0: sseg = 7'b1000000; 
            4'h1: sseg = 7'b1111001;
            4'h2: sseg = 7'b0100100; 
            4'h3: sseg = 7'b0110000;
            4'h4: sseg = 7'b0011001; 
            4'h5: sseg = 7'b0010010;
            4'h6: sseg = 7'b0000010; 
            4'h7: sseg = 7'b1111000;
            4'h8: sseg = 7'b0000000; 
            4'h9: sseg = 7'b0010000; 
            default: sseg = 7'b1111111; 
        endcase
    endfunction

    // Lógica secuencial (Memoria)
    always @(negedge clk or negedge rst_n) begin
        if (!rst_n)
            state <= S_INIT;
        else
            state <= next_state;
    end

    // Lógica combinacional 
    always @(*) begin
        case (state)
            S_INIT: if (sw == DIGITO1) next_state = S_N1;   else next_state = S_BAD;
            S_N1:   if (sw == DIGITO2) next_state = S_N2;   else next_state = S_BAD;
            S_N2:   if (sw == DIGITO3) next_state = S_N3;   else next_state = S_BAD;
            S_N3:   if (sw == DIGITO4) next_state = S_GOOD; else next_state = S_BAD;
            
            S_GOOD: next_state = S_GOOD; 
            S_BAD:  next_state = S_BAD;  
            default: next_state = S_INIT;
        endcase
    end

    // Lógica de salidas 
    always @(*) begin
        // Apagamos todos los displays por defecto
        hex3 = 7'b1111111;
        hex2 = 7'b1111111;
        hex1 = 7'b1111111;
        hex0 = 7'b1111111;

        case (state)
            S_INIT: begin
                hex0 = sseg(sw); 
            end
            S_N1: begin
                hex1 = sseg(DIGITO1); 
                hex0 = sseg(sw);      
            end
            S_N2: begin
                hex2 = sseg(DIGITO1); 
                hex1 = sseg(DIGITO2); 
                hex0 = sseg(sw);      
            end
            S_N3: begin
                hex3 = sseg(DIGITO1); 
                hex2 = sseg(DIGITO2); 
                hex1 = sseg(DIGITO3); 
                hex0 = sseg(sw);      
            end
            S_GOOD: begin
                hex3 = 7'b1000010; // G
                hex2 = 7'b0100011; // o
                hex1 = 7'b0100011; // o
                hex0 = 7'b0100001; // d
            end
            S_BAD: begin
                hex3 = 7'b1111111; // (apagado)
                hex2 = 7'b0000011; // b
                hex1 = 7'b0001000; // a
                hex0 = 7'b0100001; // d
            end
        endcase
    end

endmodule
```

### Funcionamiento
![Imagen del funcionamiento de la Práctica 4](pass1.jfif)

### Código del Testbench
```verilog
module password_tb;
    reg clk;
    reg rst_n;
    reg [3:0] sw;
    wire [6:0] hex3, hex2, hex1, hex0;

    password dut (
        .clk(clk),
        .rst_n(rst_n),
        .sw(sw),
        .hex3(hex3),
        .hex2(hex2),
        .hex1(hex1),
        .hex0(hex0)
    );

    // Reloj
    always #10 clk = ~clk;

    initial begin
        clk = 0;
        rst_n = 0;
        sw = 4'd0;
        
        #20 rst_n = 1;

        // Prueba correcta: 5 - 4 - 0 - 0
        sw = 4'd5; #20;
        sw = 4'd4; #20;
        sw = 4'd0; #20;
        sw = 4'd0; #20;
        
        #40;
        
        // Reset
        rst_n = 0; #20;
        rst_n = 1;
        
        // Prueba incorrecta: 5 - 1 - 2 - 3
        sw = 4'd5; #20;
        sw = 4'd1; #20;
        sw = 4'd2; #20;
        sw = 4'd3; #20;

        #40;
        $stop;
    end
endmodule
```

### Simulación
![Imagen de la simulación de la Práctica 4](pass2.jfif)

---

## Práctica 5: PWM

### Archivos Normales
```verilog
// Recibe un grado mediante SW, se mapea a un duty value (5% a 10%)
// Hay una frecuencia alterada para el clk, mediante un clkdiv(5MHz)
// Hay una frecuencua para clk_pwm de 50Hz
// El comparador evalua el valor de count y del valor duty para on u off
// La idea es controlar un servomotor

module pwm(
    input MAX10_CLK1_50,
    input [9:0] SW,
    output [15:0] ARDUINO_IO,
    output [0:6] HEX0,
    output [0:6] HEX1,
    output [0:6] HEX2
);

    wire rst = SW[0];
    wire [7:0] grado_raw = SW[8:1];
    reg [7:0] grado;
    wire [3:0] uni, dec, cen;

    always @(*) begin
        if (grado_raw > 8'd180)
            grado = 8'd180;
        else
            grado = grado_raw;
    end

    wire clk_div;
    reg [16:0] count = 0;
    wire [16:0] duty;
    wire arduino;

    parameter maxCount=100_000;

    clk_divider #(.FREQ (5_000_000)) clkdiv(
        .clk(MAX10_CLK1_50),
        .rst(rst), //SW 0
        .clk_div(clk_div)
    );

    //Como se mapea de duty % a grados
    assign duty = 5000 + (grado * 5000) / 180;

    // contador
    always @(posedge clk_div  or posedge rst) begin
        if(rst) count<=0;
        else if(count>=maxCount) count<=0;
        else count<=count+1;
    end

    //comparador
    assign arduino = (count < duty);
    assign ARDUINO_IO[0] = arduino;

    //displays
    assign uni = grado % 10;
    assign dec = (grado / 10) % 10;
    assign cen = (grado / 100) % 10;

    Lab2 d0 (.bcd_in(uni), .bcd_out(HEX0));
    Lab2 d1 (.bcd_in(dec), .bcd_out(HEX1));
    Lab2 d2 (.bcd_in(cen), .bcd_out(HEX2));

endmodule


module Lab2(
    input [3:0] bcd_in,
    output reg [0:6] bcd_out
);

    reg [6:0] bcd_ou;
    always @(*) begin
        case(bcd_in)
            4'b0000: bcd_ou=7'b_111_111_0;//0    
            4'b0001: bcd_ou=7'b_011_000_0;//1
            4'b0010: bcd_ou=7'b_110_110_1;//2
            4'b0011: bcd_ou=7'b_111_100_1;//3
            4'b0100: bcd_ou=7'b_011_001_1;//4
            4'b0101: bcd_ou=7'b_101_101_0;//5
            4'b0110: bcd_ou=7'b_101_111_1;//6
            4'b0111: bcd_ou=7'b_111_000_0;//7
            4'b1000: bcd_ou=7'b_111_111_1;//8
            4'b1001: bcd_ou=7'b_111_101_1;//9
            default: bcd_ou = 7'b000_000_0;
        endcase
        bcd_out= ~bcd_ou;
    end
endmodule


module clk_divider#(parameter FREQ=5_000_000)(
    input clk,
    input rst, //SW 0
    output reg clk_div
);

    reg [31:0] count;
    parameter CLK_FREQ=50_000_000;
    parameter constantNumber = CLK_FREQ/(2*FREQ);

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            count <= 32'b0;
            clk_div <= 1'b0;
        end else begin
            if (count == (constantNumber - 1)) begin
                count <= 32'b0;
                clk_div <= ~clk_div;
            end else begin
                count <= count + 1;
            end
        end
    end

endmodule
```

### Funcionamiento
![Imagen del funcionamiento de la Práctica 5](pwmn.jfif)

### Código del Testbench
```verilog
module pwm_tb;
    reg clk;
    reg [9:0] sw;
    wire [15:0] arduino_io;
    wire [0:6] hex0, hex1, hex2;

    pwm dut (
        .MAX10_CLK1_50(clk),
        .SW(sw),
        .ARDUINO_IO(arduino_io),
        .HEX0(hex0),
        .HEX1(hex1),
        .HEX2(hex2)
    );

    // Reloj
    always #10 clk = ~clk;

    initial begin
        clk = 0;
        sw = 10'd0;

        // Activar reset (SW[0] = 1)
        sw[0] = 1; 
        #40;

        // Desactivar reset y poner ángulo de 90 grados
        // Ángulo en SW[8:1]
        sw[0] = 0;
        sw[8:1] = 8'd90; 
        
        #10000;

        // Cambiar ángulo a 180 grados
        sw[8:1] = 8'd180;
        #10000;

        $stop;
    end
endmodule
```

### Simulación
![Imagen de la simulación de la Práctica 5](pwm_t.jfif)

---

## Práctica 6: UART

### Archivos Normales
```verilog
module receiver(
    input              MAX10_CLK1_50,
    input      [9:0]   SW,           
    output     [0:6]   HEX0,         
    output     [0:6]   HEX1,         
    output     [0:6]   HEX2,         
    input      [35:0]  GPIO    
);

    wire rst = SW[9];
    wire [7:0] dato_recibido;
    wire rx_ready;
    reg [7:0] dato_reg = 8'd0;

    UART_Rx #(
        .BAUD_RATE(9600),
        .CLOCK_FREQ(50000000)
    ) my_uart_rx (
        .clk(MAX10_CLK1_50),
        .rst(rst),
        .rx_in(GPIO[0]),         
        .data_out(dato_recibido),
        .ready(rx_ready)
    );
    
    always @(posedge MAX10_CLK1_50 or posedge rst) begin
        if (rst)
            dato_reg <= 8'd0;
        else if (rx_ready)
            dato_reg <= dato_recibido;
    end
    
    wire [3:0] un = dato_reg % 10;
    wire [3:0] de = (dato_reg / 10) % 10;
    wire [3:0] ce = (dato_reg / 100) % 10;

    Lab2 dec_unidades (.bcd_in(un), .bcd_out(HEX0));
    Lab2 dec_decenas  (.bcd_in(de), .bcd_out(HEX1));
    Lab2 dec_centenas (.bcd_in(ce), .bcd_out(HEX2));

endmodule


module transmiter(
    input              CLOCK_50,   
    input      [9:0]   SW,         // SW[7:0] datos, SW[9] reset
    input      [3:0]   KEY,        // KEY[0] enviar
    output     [0:6]   HEX0,       
    output     [0:6]   HEX1,       
    output     [0:6]   HEX2,       
    output     GPIO_0      
);
    
    wire rst = SW[9];
    wire start_signal = ~KEY[0];
    wire uart_busy;
    wire tx_line;
    
    wire [7:0] cuenta = SW[7:0];
    wire [3:0] un = cuenta % 10;           
    wire [3:0] de = (cuenta / 10) % 10;    
    wire [3:0] ce = (cuenta / 100) % 10;   
    
    UART_Tx #(
        .BAUD_RATE(9600),
        .CLOCK_FREQ(50000000),
        .BITS(8)
    ) my_uart_tx (
        .clk(CLOCK_50),
        .rst(rst),
        .data_in(cuenta),
        .start(start_signal),
        .tx_out(tx_line),
        .busy(uart_busy)
    );
   
    Lab2 dec_unidades (.bcd_in(un), .bcd_out(HEX0));
    Lab2 dec_decenas  (.bcd_in(de), .bcd_out(HEX1));
    Lab2 dec_centenas (.bcd_in(ce), .bcd_out(HEX2));
    
    assign GPIO_0 = tx_line;

endmodule


module UART_Rx #(parameter BAUD_RATE = 9600, parameter CLOCK_FREQ = 50000000, BITS = 8)(
    input wire clk,
    input wire rst,
    input wire rx_in,
    output reg [BITS-1:0] data_out,
    output reg data_ready
);

    localparam IDLE = 2'b00, START_BIT = 2'b01, DATA_BITS = 2'b10, STOP_BIT = 2'b11;

    reg [1:0] state;
    reg [3:0] bit_index;
    reg [15:0] baud_counter;
    reg [BITS-1:0] data_buffer;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            state <= IDLE;
            data_out <= 0;
            data_ready <= 0;
            bit_index <= 0;
            baud_counter <= 0;
        end else begin
            case (state)
                IDLE: begin
                    data_ready <= 0;
                    if (!rx_in) begin // Detecta el bit de inicio
                        state <= START_BIT;
                        baud_counter <= 0;
                    end
                end
                START_BIT: begin
                    if (baud_counter < (CLOCK_FREQ / BAUD_RATE) / 2) begin
                        baud_counter <= baud_counter + 1;
                    end else begin
                        baud_counter <= 0;
                        state <= DATA_BITS;
                        bit_index <= 0;
                    end
                end
                DATA_BITS: begin
                    if (baud_counter < CLOCK_FREQ / BAUD_RATE - 1) begin
                        baud_counter <= baud_counter + 1;
                    end else begin
                        baud_counter <= 0;
                        data_buffer[bit_index] <= rx_in; // Captura bits de datos
                        if (bit_index < BITS - 1) begin
                            bit_index <= bit_index + 1;
                        end else begin
                            state <= STOP_BIT;
                        end
                    end
                end
                STOP_BIT: begin
                    if (baud_counter < CLOCK_FREQ / BAUD_RATE - 1) begin
                        baud_counter <= baud_counter + 1;
                    end else begin
                        baud_counter <= 0;
                        data_out <= data_buffer; // Salida de datos recibidos
                        data_ready <= 1; // Indica que los datos están listos
                        state <= IDLE; // Vuelve al estado de espera para el próximo byte
                    end
                end
            endcase
        end
    end
endmodule


module UART_Tx #(parameter BAUD_RATE = 9600, parameter CLOCK_FREQ = 50000000,BITS = 8)(
    input wire clk,
    input wire rst,
    input wire [BITS-1:0] data_in,
    input wire start,
    output reg tx_out,
    output reg busy
);
    localparam IDLE = 2'b00, START_BIT = 2'b01, DATA_BITS = 2'b10, STOP_BIT = 2'b11;
    reg [1:0] state;

    reg [3:0] bit_index;
    reg [15:0] baud_counter;    
    reg [BITS-1:0] data_buffer;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            state <= IDLE;
            tx_out <= 1'b1; // Línea inactiva
            busy <= 0;
            bit_index <= 0;
            baud_counter <= 0;
        end else begin
            case (state)
                IDLE: begin
                    tx_out <= 1'b1; // Línea inactiva
                    busy <= 0;
                    if (start) begin
                        data_buffer <= data_in;
                        state <= START_BIT;
                        busy <= 1;
                    end
                end
                START_BIT: begin
                    tx_out <= 1'b0; // Bit de inicio
                    if (baud_counter < (CLOCK_FREQ / BAUD_RATE) - 1) begin
                        baud_counter <= baud_counter + 1;
                    end else begin
                        baud_counter <= 0;
                        state <= DATA_BITS;
                        bit_index <= 0;
                    end
                end
                DATA_BITS: begin
                    tx_out <= data_buffer[bit_index]; // Enviar bits de datos
                    if (baud_counter < CLOCK_FREQ / BAUD_RATE - 1) begin
                        baud_counter <= baud_counter + 1;
                    end else begin
                        baud_counter <= 0;
                        if (bit_index < BITS - 1) begin
                            bit_index <= bit_index + 1;
                        end else begin
                            state <= STOP_BIT;
                        end
                    end
                end
                STOP_BIT: begin
                    tx_out <= 1'b1; // Bit de parada
                    if (baud_counter < CLOCK_FREQ / BAUD_RATE - 1) begin
                        baud_counter <= baud_counter + 1;
                    end else begin
                        baud_counter <= 0;
                        state <= IDLE; // Volver a IDLE después del bit de parada
                    end
                end
            endcase
        end
    end
endmodule
```

### Funcionamiento
![Imagen del funcionamiento de la Práctica 6](uartn.jfif)

### Código del Testbench
```verilog
module UART_tb();

    //Señales para el transmisor
    reg clk;
    reg rst;
    reg [7:0] data_in;
    reg start;
    wire busy;

    //Señales intermedias
    wire UART_wire; // Conexión entre TX y RX

    //Señales para el receptor
    wire [7:0] data_out;
    wire data_ready;

    UART_Tx #(.BAUD_RATE(9600), .CLOCK_FREQ(50000000), .BITS(8)) UART_TX (
        .clk(clk),
        .rst(rst),
        .data_in(data_in),
        .start(start),
        .tx_out(UART_wire),
        .busy(busy)
    );

    UART_Rx #(.BAUD_RATE(9600), .CLOCK_FREQ(50000000), .BITS(8)) UART_RX (
        .clk(clk),
        .rst(rst),
        .rx_in(UART_wire),
        .data_out(data_out),
        .data_ready(data_ready)
    );

    initial begin
        clk = 0;
        rst = 0;
        data_in = 8'h00;
        start = 0;
    end

    always
        #10 clk = ~clk; // Genera un reloj de 50 MHz

    initial
    begin
        $display("Simulación iniciada");
        #30;
        rst = 1; // Activa el reset
        #10;        
        rst = 0; // Desactiva el reset
        #20000; // Espera suficiente tiempo para que el sistema se estabilice
        repeat(10) begin
            data_in = $random % 256; // Carga un dato de prueba
            start = 1; // Inicia la transmisión
            #20;
            start = 0; // Detiene la señal de inicio
            @(posedge data_ready); // Espera a que el receptor termine
            #10; // Pequeño delay para asegurar que data_out se actualizó
            $display("Dato transmitido: %h, Dato recibido: %h", data_in, data_out);
            
            // En lugar de @(negedge data_ready), esperamos a que el TX no esté ocupado
            wait(busy == 0); 
            #100; // Espacio entre transmisiones
        end
        $stop;
        $finish;
    end

    initial begin
       $dumpfile("UART_tb.vcd");
       $dumpvars(0, UART_tb);
    end

endmodule
```

### Simulación
![Imagen de la simulación de la Práctica 6](UART_t.jfif)

---

## Práctica 7: VGA

### Archivos Normales
```verilog
module hvsync_generator(
    input clk,
    input pixel_tick,
    output vga_h_sync,
    output vga_v_sync,
    output reg inDisplayArea,
    output reg [9:0] CounterX=0,
    output reg [9:0] CounterY=0
);

    reg vga_HS = 0;
    reg vga_VS = 0;

    wire CounterXmaxed = (CounterX == 799);
    wire CounterYmaxed = (CounterY == 524);

    always @(posedge clk) begin
        if (pixel_tick) begin
            if (CounterXmaxed) CounterX <= 0;
            else CounterX <= CounterX + 1;
        end
    end

    always @(posedge clk) begin
        if (pixel_tick && CounterXmaxed) begin
            if (CounterYmaxed) CounterY <= 0;
            else CounterY <= CounterY + 1;
        end
    end

    always @(posedge clk) begin
        if (pixel_tick)
            vga_HS <= (CounterX >= (640 + 16) && CounterX < (640 + 16 + 96));
    end

    always @(posedge clk) begin
        if (pixel_tick)
            vga_VS <= (CounterY >= (480 + 10) && CounterY < (480 + 10 + 2));
    end

    always @(posedge clk) begin
        if (pixel_tick)
            inDisplayArea <= (CounterX < 640) && (CounterY < 480);
    end

    assign vga_h_sync = ~vga_HS;
    assign vga_v_sync = ~vga_VS;

endmodule


module VGADemo(
    input MAX10_CLK1_50,
    output [3:0] vga_red,
    output [3:0] vga_green,
    output [3:0] vga_blue,
    output hsync_out,
    output vsync_out
);

    wire inDisplayArea;
    wire [9:0] CounterX;
    wire [9:0] CounterY;

    // Generador de 25MHz
    reg pixel_tick = 0;
    always @(posedge MAX10_CLK1_50)
        pixel_tick <= ~pixel_tick;

    hvsync_generator hvsync(
        .clk(MAX10_CLK1_50),
        .pixel_tick(pixel_tick),
        .vga_h_sync(hsync_out),
        .vga_v_sync(vsync_out),
        .CounterX(CounterX),
        .CounterY(CounterY),
        .inDisplayArea(inDisplayArea)
    );

    // Lógica de ajedrez: cuadros de 64x64 píxeles
    wire is_white = CounterX[6] ^ CounterY[6];

    // Asignamos 4'b1111 (brillo máximo) si es blanco, 4'b0000 si es negro
    assign vga_red   = (inDisplayArea && is_white) ? 4'b1111 : 4'b0000;
    assign vga_green = (inDisplayArea && is_white) ? 4'b1111 : 4'b0000;
    assign vga_blue  = (inDisplayArea && is_white) ? 4'b1111 : 4'b0000;

endmodule
```

### Funcionamiento
![Imagen del funcionamiento de la Práctica 7](Ajedrez.jfif)

### Código del Testbench
```verilog
module VGADemo_tb;
    reg clk;
    wire [3:0] vga_red;
    wire [3:0] vga_green;
    wire [3:0] vga_blue;
    wire hsync_out;
    wire vsync_out;

    VGADemo dut (
        .MAX10_CLK1_50(clk),
        .vga_red(vga_red),
        .vga_green(vga_green),
        .vga_blue(vga_blue),
        .hsync_out(hsync_out),
        .vsync_out(vsync_out)
    );

    // Reloj
    always #10 clk = ~clk;

    initial begin
        clk = 0;
        
        // Esperar suficiente tiempo para ver movimiento de señales
        #50000;
        
        $stop;
    end
endmodule
```

### Simulación
![Imagen de la simulación de la Práctica 7](VGA_t.jfif)