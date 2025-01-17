# ExUnit Cluster

NOTE: Requires at least OTP 25 (which is first supported by elixir 1.13.4).

Spin up dynamic clusters in ExUnit tests with no special setup necessary.

You can run normal tests, alongside clustered tests.

To run distributed tests simply start a `ExUnitCluster.Manager` in a test module
```elixir
defmodule MyTest do
  use ExUnit.Case, async: true

  setup ctx do
    cluster = start_supervised!({ExUnitCluster.Manager, ctx})
    [cluster: cluster]
  end

  test "I can spin up a 3 node cluster", ctx do
    n1 = ExUnitCluster.start_node(ctx.cluster)
    n2 = ExUnitCluster.start_node(ctx.cluster)
    n3 = ExUnitCluster.start_node(ctx.cluster)

    nodes = Cluster.get_nodes(ctx.cluster)

    assert Enum.sort([n1, n2, n3]) == Enum.sort(nodes)

    res =
      Enum.flat_map(nodes, fn n ->
        ExUnitCluster.call(ctx.cluster, n, Node, :list, [[:visible, :this]])
      end)

    assert length(res) == 9
    assert MapSet.size(MapSet.new(res)) == 3
  end
end
```

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `ex_unit_cluster` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:ex_unit_cluster, "~> 0.1.0"}
  ]
end
```

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at <https://hexdocs.pm/ex_unit_cluster>.
