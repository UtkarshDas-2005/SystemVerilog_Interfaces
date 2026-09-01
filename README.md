# SystemVerilog_Interfaces
---

## 🎯 Objective

Learn how to group related signals into a single interface and use it to connect the **DUT and Testbench**.

---

## 1. What is an Interface?

An **interface** is a SystemVerilog construct used to group multiple related signals into one block.

Instead of connecting many signals individually between DUT and testbench, we can put them inside an interface.

### Without Interface

```systemverilog
module dut (
    input  clk,
    input  reset,
    input  req,
    input  [7:0] data,
    output ack
);
```

Many signals have to be connected separately.

### With Interface

```systemverilog
interface bus_if;
    logic clk;
    logic reset;
    logic req;
    logic [7:0] data;
    logic ack;
endinterface
```

Now all signals are grouped together.

---

## 2. Basic Syntax

```systemverilog
interface interface_name;

    // signals
    logic clk;
    logic reset;
    logic req;
    logic ack;

endinterface
```

---

## 3. Why Do We Use Interfaces?

Interfaces provide:

* Better code organization
* Easier DUT–testbench connection
* Reduced number of ports
* Reusability
* Easier verification
* Support for **modports**
* Support for **clocking blocks**
* Very useful in **UVM**

---

## 4. Complete Example

### Interface

```systemverilog
interface bus_if;

    logic clk;
    logic reset;
    logic req;
    logic [7:0] data;
    logic ack;

endinterface
```

### DUT

```systemverilog
module dut(bus_if bus);

    always_ff @(posedge bus.clk) begin

        if (bus.reset)
            bus.ack <= 0;

        else if (bus.req)
            bus.ack <= 1;

        else
            bus.ack <= 0;

    end

endmodule
```

### Testbench

```systemverilog
module tb;

    bus_if bus();

    dut d1(bus);

    initial begin

        bus.clk   = 0;
        bus.reset = 1;
        bus.req   = 0;
        bus.data  = 8'h00;

        #10;
        bus.reset = 0;

        #10;
        bus.req = 1;
        bus.data = 8'hAA;

        #10;
        bus.req = 0;

        #20;
        $finish;

    end

    always #5 bus.clk = ~bus.clk;

endmodule
```

---

# 5. Interface Instance

An interface is instantiated similarly to a module.

```systemverilog
bus_if bus();
```

Here:

* `bus_if` → interface type
* `bus` → interface instance
* `()` → instance declaration

---

# 6. Modport

A **modport** defines which signals can be read or written by a particular component.

This helps control the direction of signals.

### Example

```systemverilog
interface bus_if;

    logic clk;
    logic req;
    logic ack;
    logic [7:0] data;

    modport DUT (
        input clk,
        input req,
        input data,
        output ack
    );

    modport TB (
        input clk,
        input ack,
        output req,
        output data
    );

endinterface
```

Now DUT and Testbench have different access directions.

---

# 7. Using Modport in DUT

```systemverilog
module dut(bus_if.DUT bus);

    always_ff @(posedge bus.clk) begin

        if (bus.req)
            bus.ack <= 1;

        else
            bus.ack <= 0;

    end

endmodule
```

---

# 8. Using Modport in Testbench

```systemverilog
module tb;

    bus_if bus();

    dut d1(bus.DUT);

endmodule
```

The modport provides a clean connection between the DUT and testbench.

---

# 9. Interface with Clocking Block

A **clocking block** is used to synchronize testbench signal sampling and driving with a clock.

```systemverilog
interface bus_if(input logic clk);

    logic req;
    logic ack;
    logic [7:0] data;

    clocking cb @(posedge clk);

        output req;
        output data;

        input ack;

    endclocking

endinterface
```

This is very useful in verification environments.

---

# 10. Interface + Clocking Block Example

```systemverilog
interface bus_if(input logic clk);

    logic req;
    logic ack;
    logic [7:0] data;

    clocking cb @(posedge clk);
        output req;
        output data;
        input ack;
    endclocking

endinterface
```

Testbench can access signals through:

```systemverilog
bus.cb.req
bus.cb.data
bus.cb.ack
```

---

# 11. Interface vs Module

| Module                               | Interface                       |
| ------------------------------------ | ------------------------------- |
| Mainly used to describe hardware     | Mainly used for communication   |
| Has ports                            | Groups related signals          |
| Used to implement DUT                | Used to connect DUT/TB          |
| Defines hardware functionality       | Defines communication structure |
| Cannot directly replace an interface | Very useful in verification     |

---

# 12. Interface in UVM

Interfaces are extremely important in UVM.

Typical connection:

```text
             TESTBENCH
                 |
             UVM DRIVER
                 |
                 ↓
          SYSTEMVERILOG
            INTERFACE
                 |
                 ↓
                DUT
```

The UVM driver drives DUT signals through the virtual interface.

Example:

```systemverilog
virtual bus_if vif;
```

Later, the driver can access:

```systemverilog
vif.req
vif.data
vif.ack
```

---

# 🧠 Key Points

* Interface groups related signals.
* It simplifies DUT–testbench connections.
* `modport` defines signal directions/access.
* `clocking block` synchronizes TB activity with the clock.
* Interfaces are heavily used in **verification and UVM**.
* A UVM driver commonly accesses an interface through a **virtual interface**.

---

