Extending the model
===================

Adding architectural state
--------------------------

Adding registers (such as for floating-point) would involve naming
them and defining their read and write accessors, as is done for the
integer registers in `riscv_types.sail`.  For modularity, these new
definitions can be added in a separate file.  If these registers have
properties of control-and-status registers (CSRs), or depend on
privilege level (such as hypervisor-mode registers), additional access
control checks would need to be provided as is done for the standard
CSRs in `riscv_sys_regs.sail` and `riscv_sys_control.sail`.  In addition,
the bits `mstatus.XS` and `mstatus.SD` may need to be updated or
extended to handle any extended register state.

Adding a new privilege level or functionality restricted by privilege
level will normally be accompanied by defining new exception causes
and their encodings.  This will require modifying and extending the
existing definitions for privilege levels and exceptions in
`riscv_types.sail`, and modifying the exception handling and privilege
transition functions in `riscv_sys_control.sail`.

Adding low-level platform functionality
---------------------------------------

Adding support for new devices such as interrupt controllers and
similar memory-mapped I/O (MMIO) entities strictly falls outside the
purview of the formal model itself, and typically is not done
directly in the Sail model.  However, bindings to this external
functionality can be provided to Sail definitions using the `extern`
construct of the Sail language. `riscv_platform.sail` can be examined
to see how this is done for the SiFive core-local interrupt (CLINT)
controller, the HTIF timer and terminal devices.  The
implementation of the actual functionality provided by these MMIO
devices would need to be added to the C and OCaml emulators.

If this functionality requires the definition of new interrupt
sources, their encodings would need to be added to `riscv_types.sail`,
and their delegation and handling added to `riscv_sys_control.sail`.

Modifying memory access
-----------------------

Physical memory addressing and access is defined in `riscv_mem.sail`.
Any new types of memory (such as scratchpad, tags, or MMIO device
memory) that are accessible via physical addresses will require
modifying the `mem_read`, `mem_write_value` or their supporting
functions `checked_mem_read` and `checked_mem_write`.

The actual content of such memory, and its modification, can be
defined in separate sail files.  This functionality will have access
to any newly defined architectural state.  One can examine how normal
physical memory access is implemented in `riscv_mem.sail` with helpers
in `prelude.sail`.

Virtual memory is implemented in `riscv_vmem.sail`, and defining new
address translation schemes will require modifying the
top-level `translateAddr` function.  Any access control checks on
virtual addresses and the specifics of the new address translation can be
specified in a separate file.  This functionality can access any newly
defined architectural state.

Adding new instructions
-----------------------

This is typically simpler than adding new architectural state or
memory interposition.  Each new set of instructions can be specified
in a separate self-contained file, with their instruction encodings,
assembly language specifications and the corresponding encoders and
decoders, and execution semantics. `riscv.sail` can be examined for
examples on how this can done.  These instruction definitions can
access any newly defined architectural state and perform virtual or
physical memory accesses as is done in `riscv.sail`.

General guidelines
------------------

For any new extension, it is helpful to factor it out into the above
items.  When specifying and implementing the extension, it is expected
to be easier to implement it in the above listed order.

Example
-------

As an example, one can examine the implementation of the 'N' extension
for user-level interrupt handling.  The architectural state to support
'N' is specified in `riscv_next_regs.sail`, added control
functionality is in `riscv_next_control.sail`, and added instructions
are in `riscv_insts_next.sail`.  In addition, privilege transition and
interrupt delegation logic in `riscv_sys_control.sail` has been
extended.
