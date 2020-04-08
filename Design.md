# Arrow Compression

## Overview
The arrow format stores column attributes in contiguous, aligned memory addresses. In contrast to delta-chain version storage, no pointers are allowed. Because of this, Arrow compression increases the performance of reading (scanning) tuples in a block with the trade-off of increasing the cost of writing (inserting, updating, deleting) tuples in that block. To maximize the benefit of arrow compression while minimizing its cost, block compaction (the process of taking blocks that are not in arrow format and manipulating them so that they are in arrow format) targets blocks that are not as likely to be written to. Block compaction moves tuples so that they are contiguous in memory and then stores each tuple attribute in the arrow format, thereby decreasing the amount of memory used in two ways. While the arrow format and block compaction are currently implemented in the system, their use is not. The 75%, 100%, and 125% goals of our component are: to add index updates to the block compaction process, to support the ability of the execution engine to operate on compressed data, and to provide a compaction policy (when block compaction should occur and on what blocks) that allows arrow compression to fully benefit the system.

## Scope
This feature relies upon the following files (.h/.cpp is left out for convenience):
- storage_defs: defines tuple slot, block, etc.
- arrow_block_metadata: defines arrow ‘metadata’ (format) of arrow blocks.
- tuple_access_strategy: determines tuple access in a data table block based on block format (arrow or not arrow).
- block_access_controller: determines reader/writer access to block based on block status (hot/cooling/cold/frozen).
- block_compactor: implements block compaction logic to make data contiguous.
- access_observer: adds blocks to a compaction queue (part of garbage collection).

This feature modifies the following files (.h/.cpp is left out for convenience):
- data_table: adds an InsertIntoFreezingBlock that inserts a tuple into a specific block where the specific block is known to be in the 'Freezing' state and therefore not accessible to readers or writers. This function is intended to only be used only by the block compaction process.
- block_compactor: modifies the MoveTuple function to update the indexes for every tuple that is moved to a new tuple slot.
- ** Potentially ** sql_table, storage_interface, bytecode_handlers, bytecodes, vm, bytecode_generator, builtins, and 
sema_builtin: this is the chain of files that need to be modified in order to add a built-in that can be utilized by the Terrier Processing Language (TPL).
- ** Potentially ** other execution engine layer files: additions that are needed to modify the Terrier Processing Language (TPL) API in order to allow compressed data to be operated on.

## Glossary (Optional)
As of now, we have not added any unintuitive concepts or names of components.

## Architectural Design
>Explain the input and output of the component, describe interactions and breakdown the smaller components if any. Include diagrams if appropriate.

## Design Rationale
>Explain the goals of this design and how the design achieves these goals. Present alternatives considered and document why they are not chosen.

## Testing Plan
First, we would like to establish the baseline performance for OLAP queries by running a standard OLAP workload like TPC-H on terrier. With this information in hand, we can measure the speedup that our implementation provides for OLAP queries.

We would then enable the compaction process, verify the correctness of the arrow format blocks and then run the same OLAP queries on these compressed blocks. At this stage, the reader will still be materializing the tuples to be read and so we do not expect any performance improvements here. 

Our final testing step would be to run the same OLAP queries directly on the compressed data without materializing tuples to be read. We expect a significant performance improvement here because there is no overhead of materialising tuples and also that tuples within a block are present in contiguous memory locations. 

If we get to the 125% project goal (designing policy for marking a block for conversion), we can also run rigorous tests with different policies on hybrid workloads. This way, we can determine the best policy for marking a block for compaction.

## Trade-offs and Potential Problems
>Write down any conscious trade-off you made that can be problematic in the future, or any problems discovered during the design process that remain unaddressed (technical debts).

## Future Work
Currently we are working on our 75% goal which is to enable compression of blocks through the execution layer (updates tables as well as indexes). We are yet to start working on the execution layer but we think we will have to write TPL code which will be called as a builtin from the `block_compactor` code. 

For our 100% goal, we will be running OLAP queries directly on top of the compressed data and so, we will have to add functionality for reading FROZEN blocks directly rather than materializing them. 

For the 125% goal we have to experiment with the different policies for starting the cooling process. This would involve changes to the policy in the garbage collector code.
