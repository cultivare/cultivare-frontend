<template>
    <v-container>
        <div ref="network" style="height: 50em; background-color: #2a2a2a;"></div>
        <v-slider class="pr-4 pt-3" v-model.number="depthLimit" label="Depth Limit" :min="1" :max="6"
            show-ticks="always" step="1" tick-size="3"></v-slider>
    </v-container>
</template>

<script>
import { Network } from 'vis-network';
import { DataSet } from 'vis-data';
import api from '@/services/api';

export default {
    name: 'GraphComponent',
    props: {
        culture_id: {
            type: [String, Number],
            required: true,
        },
    },
    data() {
        return {
            network: null,
            nodes: new DataSet(),
            edges: new DataSet(),
            depthLimit: 2,
        };
    },
    watch: {
        culture_id: {
            handler() {
                this.recreateNetwork();
            },
            immediate: true,
        },
        depthLimit: {
            handler() {
                this.recreateNetwork();
            },
        },
    },
    beforeUnmount() {
        this.destroyNetwork();
    },
    methods: {
        async recreateNetwork() {
            this.destroyNetwork(); // Destroy the old network if it exists
            await this.fetchAndRenderGenealogy(); // Fetch data and render
        },
        destroyNetwork() {
            if (this.network !== null) {
                this.network.destroy();
                this.network = null;
            }
        },
        async fetchAndRenderGenealogy() {
            try {
                const response = await api.get(
                    `/cultures/${this.culture_id}/genealogy?depth_limit=${this.depthLimit}`
                );
                const formattedData = this.formatGenealogyData(response.data);
                this.nodes.clear();
                this.edges.clear();
                this.nodes.add(formattedData.nodes.get()); // Use add instead of update for a clean start
                this.edges.add(formattedData.edges.get());

                // Create a new network if it doesn't exist
                if (!this.network) {
                    this.renderTree({ nodes: this.nodes, edges: this.edges });
                } else {
                    // This should never be reached now, as we are always recreating
                    this.network.setData({
                        nodes: this.nodes,
                        edges: this.edges
                    });
                }

                // Select and focus after data is set and the network is (re)created
                this.network.fit({ nodes: [this.culture_id] });
                this.network.focus(this.culture_id, {
                    scale: 5,
                    animation: false,
                });

            } catch (error) {
                console.error('Error fetching or rendering genealogy:', error);
            }
        },
        formatGenealogyData(data) {
            const nodes = new DataSet();
            const edges = new DataSet();

            data.forEach(item => {
                if (!nodes.get(item.id)) {
                    nodes.add({
                        id: item.id,
                        label: item.name,
                        route: `/cultures/${item.id}`,
                    });
                }

                if (item.parent_ids && item.parent_ids.length > 0) {
                    item.parent_ids.forEach(parentId => {
                        if (!edges.get({ filter: edge => edge.from === parentId && edge.to === item.id }).length) {
                            edges.add({
                                from: parentId,
                                to: item.id,
                            });
                        }
                    });
                }
            });

            return { nodes, edges };
        },
        renderTree(data) {
            const container = this.$refs.network;
            const options = {
                autoResize: true,
                height: '100%',
                width: '100%',
                physics: {
                    barnesHut: {
                        centralGravity: 0.1,
                        springLength: 200 // Default edge length will be 150 pixels
                    }
                },
                edges: {
                    // length: 150,
                    arrows: {
                        to: {
                            enabled: true,
                            scaleFactor: 1,
                            type: 'arrow',
                        },
                    },
                },
                nodes: {
                    shape: 'box',
                    color: {
                        background: '#D2E5FF',                        
                        border: '#2B7CE9',
                        highlight: {
                            background: '#D2E5FF',
                            border: '#2B7CE9',
                        },
                    },
                    font: {
                        color: '#343434',
                        size: 14,
                        face: 'arial',
                        background: 'none',
                    },
                },
            };

            this.network = new Network(container, data, options);

            this.network.on('selectNode', params => {
                if (params.nodes.length > 0) {
                    const nodeId = params.nodes[0];
                    const node = this.nodes.get(nodeId);

                    // Update node colors
                    this.nodes.forEach(node => {
                        this.nodes.update({
                            id: node.id,
                            color: { background: '#D2E5FF' },
                        });
                    });
                    this.nodes.update({
                        id: nodeId,
                        color: { background: 'orange' },
                    });

                    if (node.route) {
                        this.$router.push(node.route);
                    }
                }
            });

            this.network.on('deselectNode', () => {
                // Reset all node colors when deselected
                this.nodes.forEach(node => {
                    this.nodes.update({
                        id: node.id,
                        color: { background: '#D2E5FF' },
                    });
                });
                // Highlight the current culture node
                this.nodes.update({
                    id: this.culture_id,
                    color: { background: 'orange' },
                });
            });

            // Initial highlight of the current culture node
            this.network.on("afterDrawing", () => {
                this.nodes.update({
                    id: this.culture_id,
                    color: { background: 'orange' },
                });
            });
        },
    },
};
</script>