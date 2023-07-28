<template>
    <div :id="'heatmap-container' + id" class="heatmap-container" :style="containerStyle" ref="heatmapContainer">
        <!-- Render buttons based on the transformed interactions data -->
        <button v-for="(dataPoint, index) in transformedData" :key="index" class="heatmap-button"
            :style="buttonStyle(dataPoint)" disabled="true">
        </button>
    </div>
</template>
  
<script>
export default {
    props: {
        screenResolution: {
            type: String,
            required: true,
        },
        data: {
            type: Array,
            default: () => [],
            required: true,
        },
        id: {
            type: String,
            required: true,
        },
    },
    mounted() {
        this.$nextTick(() => {
            const containerElement = this.$refs.heatmapContainer;
            const containerRect = containerElement.getBoundingClientRect();
            this.containerTop = containerRect.top + window.scrollY;
            this.containerLeft = containerRect.left + window.scrollX;
        });
    },
    computed: {
        containerStyle() {
            // Parse the screen resolution to get width and height
            const [width, height] = this.screenResolution.split("x").map((val) => parseInt(val.trim()));
            const scaledWidth = width / 2;
            const scaledHeight = height / 2;
            return {
                width: `${scaledWidth}px`, // Half of the original width
                height: `${scaledHeight}px`, // Half of the original height
                position: "relative", // Required for absolute positioning of buttons
                border: "1px solid #ccc", // Optional: Add a border to the heatmap container
            };
        },
        transformedData() {
            // Calculate the scaling factor to fit the heatmap within the container
            const scalingFactor = 0.5; // Half of the original size

            return this.data.map((dataPoint) => {
                const [x, y] = dataPoint.location.split("x").map((val) => parseFloat(val.trim()));

                // Scale and position the button within the container
                const scaledX = x/2;
                const scaledY = y/2;

                const buttonLeft = scaledX / this.containerWidth * 100; // Calculate the button position relative to the top-left corner of the container
                const buttonTop = scaledY / this.containerHeight * 100;

                // Calculate the button position relative to the top-left corner of the container

                return { location: { x: buttonLeft, y: buttonTop }, interaction: dataPoint.count };
            });
        },
        originalWidth() {
            const [width] = this.screenResolution.split("x").map((val) => parseInt(val.trim()));
            return width;
        },
        originalHeight() {
            const [, height] = this.screenResolution.split("x").map((val) => parseInt(val.trim()));
            return height;
        },
        containerWidth() {
            return this.originalWidth / 2; // Half of the original width
        },
        containerHeight() {
            return this.originalHeight / 2; // Half of the original height
        },
    },
    data() {
        return {
            containerTop: 0,
            containerLeft: 0,
        };
    },
    methods: {
        buttonStyle(dataPoint) {
            const buttonSize = 20; // Adjust the button size as needed
            const color = this.getColor(dataPoint.interaction);
            return {
                position: "absolute",
                top: `${dataPoint.location.y}%`, // Half of the original Y location
                left: `${dataPoint.location.x}%`, // Half of the original X location
                transform: "translate(-50%, -50%)", // Center the button at the location
                width: `${buttonSize}px`, // Set the button width
                height: `${buttonSize}px`, // Set the button height
                borderRadius: "50%", // Make the button round
                background: color,
                color: "#fff", // Optional: Set the text color inside the button
                fontSize: "12px", // Optional: Adjust font size for the interaction number
                fontWeight: "bold",
                border: "none", // Optional: Remove button border
                cursor: "pointer", // Optional: Add cursor pointer on hover
            };
        },
        getColor(interaction) {
            const maxInteraction = Math.max(...this.data.map((dataPoint) => dataPoint.count));
            const minInteraction = Math.min(...this.data.map((dataPoint) => dataPoint.count));

            if (interaction <= minInteraction) {
                return "#7BDCB5"; // Light green for the minimum interaction
            } else if (interaction <= maxInteraction) {
                // Calculate the percentage of interaction relative to the maxInteraction
                const percentage = (interaction - minInteraction) / (maxInteraction - minInteraction);

                // Calculate the RGB value based on the percentage
                const r = Math.floor(124 + (255 - 124) * percentage);
                const g = Math.floor(220 + (215 - 220) * percentage);
                const b = Math.floor(181 + (71 - 181) * percentage);

                return `rgb(${r},${g},${b})`; // Dynamic color based on the interaction
            } else {
                return "#FF6347"; // Tomato for the maximum interaction
            }
        },
    },
};
</script>
  
<style scoped>
/* Adjust styles for the heatmap container and buttons */
.heatmap-container {
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #faf8fb;
}

.heatmap-button {
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    outline: none;
}
</style>
  