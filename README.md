# cpu-instruction-counter

Measuring CPU instructions using Linux Performance Counters / `perf_event_open()`.

Especially useful for writing **performance regression tests**.

Counting CPU instructions counts is a measure of performance that is largely independent of machine/processor types and current load, thus making for more stable benchmarks than measuring wall time or system CPU time.

## Example usage

```haskell
(result, instrs) <- withInstructionsCounted $ do
  return () -- your code here

print (result, instructions)
```

## Example regression test

Put something like this into your CI to get notified about performance regressions in:

* your code
* libraries you use
* code generated by the compiler

```haskell
import           System.CPUInstructionCounter

main :: IO ()
main = do

  (_, sumInstrs) <- withInstructionsCounted $ do
    return $! sum [1..10000::Int]

  let expected = 90651 -- measured on my Core i5-4590, with lts-10.3

  if sumInstrs > (2 * expected)
    then error $ "sum [1..10000] performance regressed; took " ++ show sumInstrs ++ " instructions"
    else putStrLn $ "sum [1..10000] performance OK; took " ++ show sumInstrs ++ " instructions"
```


## Root permissions

`perf_event_open()` might be restricted on your system. If `cat /proc/sys/kernel/perf_event_paranoid` shows a number greater than `2`, you'll need `sudo` to run the above examples.

Alternatively, run `echo 2 | sudo tee /proc/sys/kernel/perf_event_paranoid`.

On CI systems under your control and many public CI services, there should be no barriers to root access so this should not be a showstopper to using this approach for performance regression tests.
