<script setup lang="ts">
  import * as vNG from "v-network-graph";
  import { ForceLayout } from "v-network-graph/lib/force-layout";
  import { reactive } from "vue";

  const numberOfNodes = 30;
  const numberOfEdges = 10;
  const startingNode = 0;
  const targetNode = 1;

  const configs = reactive(
    vNG.defineConfigs({
      view: {
        scalingObjects: true,
        layoutHandler: new ForceLayout(),

        // layoutHandler: new ForceLayout({
        //   positionFixedByDrag: false,
        //   positionFixedByClickWithAltKey: true,
        // }),
      },
      node: {
        label: {
          visible: true,
          fontFamily: undefined,
          fontSize: 11,
          lineHeight: 1.1,
          color: "#FFFFFF",
          margin: 4,
          direction: "south",
          text: (node) => {
            return node[0];
          },
        },
      },
      edge: {
        normal: {
          width: (edge) => edge.width ?? 1,
          color: (edge) => edge.color ?? "green",
        },
        marker: {
          source: {
            type: "none",
            width: 4,
            height: 4,
            margin: -1,
            units: "strokeWidth",
            color: null,
          },
          target: {
            type: "arrow",
            width: 4,
            height: 4,
            margin: -1,
            units: "strokeWidth",
            color: null,
          },
        },
      }

    })
  );

  const allNodes = new Array(numberOfNodes)
    .fill("")
    .map((_, idx) => ({ name: idx + "" }));

  const nodes = Object.entries(allNodes.map((node) => [node.name, node]));

  const seeds = allNodes.slice(2);

  function getRandomSeed(source: string) {
    const seedsWithoutSource = seeds.filter((s) => s.name !== source);
    return seedsWithoutSource[
      Math.floor(Math.random() * seedsWithoutSource.length)
    ];
  }
  
  const edges = reactive({
    edge0: {source: "4", target: "0"},
    edge1: {source: "6", target: "1"},
    edge2: {source: "0", target: "4"},
    edge3: {source: "1", target: "6"},

    // only have 2 edges per node
    // edge4: {source: "0", target: "1"},
    // edge5: {source: "0", target: "2"},

    ...Object.fromEntries(
      seeds.flatMap((node) => {
        return seeds
          .filter((s) => s.name !== node.name)
          .slice(0, numberOfEdges / 2)
          .map((other) => {
            return [
              `seed-${node.name}-${other.name}`,
              { source: node.name, target: other.name },
            ]
          })
      })
    ),
  });

  const seen = new Set();

  async function lookup(startNode: string, to: string) {
    if (seen.has(startNode)) {
      return;
    }

    seen.add(startNode);

    if (startNode === to) {
      return true;
    }

    const neighbours = Object.entries(edges).filter(
      ([, { source }]) => source === startNode
    );

    const results = await Promise.all(
      neighbours.map((neighboor) => {
        neighboor[1].color = "yellow"
        return new Promise (resolve => {
          setTimeout(() => {
            resolve(lookup(neighboor[1].target, to))
          }, 4000)
        }).then((found) => {
          if (found) {
            (edges as any)[`edge` + Math.random()] = {
              target: to,
              source: startNode,
            };
            return true;
          }
        })
      })
    );
    
    return results.some(Boolean);
  }

  // Starting at node 0, find node 1 (or whatever we define)
  lookup(startingNode + "", targetNode + "");
</script>

<template>
  <v-network-graph
    class="graph"
    :nodes="nodes"
    :edges="edges"
    :configs="configs"
    :zoom-level="2"
  />
</template>

<style>
.graph {
  width: 99.4vw;
  height: 99.3vh;
  border: 1px transparent solid;
}
</style>