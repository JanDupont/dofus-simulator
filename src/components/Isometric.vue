vue
<template>
	<div id="app-container">
		<div id="bg-image"></div>

		<canvas id="isocanvas"></canvas>
		<div class="HUD">
			<div class="HUD-left"></div>
			<div class="HUD-right">
				<v-container><v-btn id="toggleCoordinates">toggle Coordinates</v-btn></v-container>
			</div>
		</div>
	</div>
</template>

<script>
import { IsometricMap } from "./map.js";
export default {
	data() {
		return {
			tileColumnOffset: 100, //pixels
			tileRowOffset: 50, //pixels
			pageNavBarOffset: 64, //pixels

			originX: 0, //offset from left
			originY: 0, //offset from top

			Xtiles: 0, //Number of tiles in X-dimension
			Ytiles: 0, //Number of tiles in Y-dimension

			selectedTileX: -1,
			selectedTileY: -1,

			context: undefined,
			canvas: undefined,

			tileImages: [],

			showCoordinates: false,
		};
	},
	methods: {
		load() {
			this.tileImages = new Array();
			var loadedImages = 0;
			var totalImages = IsometricMap.tiles.length;

			// Load all the images before we run the app
			var self = this;
			for (var i = 0; i < IsometricMap.tiles.length; i++) {
				this.tileImages[i] = new Image();
				this.tileImages[i].onload = function () {
					if (++loadedImages >= totalImages) {
						self.run();
					}
				};
				this.tileImages[i].src = IsometricMap.tiles[i];
			}
		},
		run() {
			this.canvas = $("#isocanvas");
			this.context = this.canvas[0].getContext("2d");

			this.Xtiles = IsometricMap.map.length;
			this.Ytiles = IsometricMap.map[0].length;

			var self = this;
			$(window).on("resize", function () {
				self.updateCanvasSize();
				self.redrawTiles();
			});

			// gets the selected tile
			$(window).on("mousemove", function (e) {
				e.pageX = e.pageX - self.tileColumnOffset / 2 - self.originX;
				e.pageY = e.pageY - self.tileRowOffset / 2 - self.originY;
				let tileX = Math.round(
					e.pageX / self.tileColumnOffset - (e.pageY - self.pageNavBarOffset) / self.tileRowOffset
				);
				let tileY = Math.round(
					e.pageX / self.tileColumnOffset + (e.pageY - self.pageNavBarOffset) / self.tileRowOffset
				);

				self.selectedTileX = tileX;
				self.selectedTileY = tileY;

				self.redrawTiles();
			});

			// on button click
			$(toggleCoordinates).on("click", function () {
				self.showCoordinates = !self.showCoordinates;
				self.redrawTiles();
			});

			// on window click
			$(window).on("click", function (e) {
				if (self.isCursorOnMap()) {
					IsometricMap.map[self.selectedTileX][self.selectedTileY] = 1;
				}

				self.redrawTiles();
			});

			this.updateCanvasSize();
			this.redrawTiles();
		},

		updateCanvasSize() {
			var width = $(window).width();
			var height = $(window).height();

			this.context.canvas.width = width;
			this.context.canvas.height = height;

			this.originX = width / 2 - (this.Xtiles * this.tileColumnOffset) / 2;
			this.originY = height / 2;
		},

		redrawTiles() {
			this.context.canvas.width = this.context.canvas.width;

			for (var Xi = this.Xtiles - 1; Xi >= 0; Xi--) {
				for (var Yi = 0; Yi < this.Ytiles; Yi++) {
					this.drawTile(Xi, Yi);
				}
			}

			this.drawDiamond(this.selectedTileX, this.selectedTileY, "yellow");
			if (this.showCoordinates && this.isCursorOnMap()) {
				this.context.fillStyle = "yellow";
				var idx = IsometricMap.map[this.selectedTileX][this.selectedTileY];
				this.context.font = "14pt Arial";
				this.context.fillText(IsometricMap.tiles[idx].replace("/assets/tiles/", ""), 20, 30);
			}
		},
		isCursorOnMap() {
			return (
				this.selectedTileX >= 0 &&
				this.selectedTileX < this.Xtiles &&
				this.selectedTileY >= 0 &&
				this.selectedTileY < this.Ytiles
			);
		},
		drawTile(Xi, Yi) {
			var offX = (Xi * this.tileColumnOffset) / 2 + (Yi * this.tileColumnOffset) / 2 + this.originX;
			var offY = (Yi * this.tileRowOffset) / 2 - (Xi * this.tileRowOffset) / 2 + this.originY;

			var imageIndex = IsometricMap.map[Xi][Yi];
			this.context.drawImage(this.tileImages[imageIndex], offX, offY);

			if (this.showCoordinates) {
				this.context.fillStyle = "orange";
				this.context.fillText(
					Xi + ", " + Yi,
					offX + this.tileColumnOffset / 2 - 9,
					offY + this.tileRowOffset / 2 + 3
				);
			}
		},
		drawDiamond(Xi, Yi, color) {
			if (this.isCursorOnMap()) {
				var offX = (Xi * this.tileColumnOffset) / 2 + (Yi * this.tileColumnOffset) / 2 + this.originX;
				var offY = (Yi * this.tileRowOffset) / 2 - (Xi * this.tileRowOffset) / 2 + this.originY;

				this.drawLine(offX, offY + this.tileRowOffset / 2, offX + this.tileColumnOffset / 2, offY, color);
				this.drawLine(
					offX + this.tileColumnOffset / 2,
					offY,
					offX + this.tileColumnOffset,
					offY + this.tileRowOffset / 2,
					color
				);
				this.drawLine(
					offX + this.tileColumnOffset,
					offY + this.tileRowOffset / 2,
					offX + this.tileColumnOffset / 2,
					offY + this.tileRowOffset,
					color
				);
				this.drawLine(
					offX + this.tileColumnOffset / 2,
					offY + this.tileRowOffset,
					offX,
					offY + this.tileRowOffset / 2,
					color
				);
			}
		},
		drawLine(x1, y1, x2, y2, color) {
			color = typeof color !== "undefined" ? color : "white";
			this.context.strokeStyle = color;
			this.context.beginPath();
			this.context.lineWidth = 1;
			this.context.moveTo(x1, y1);
			this.context.lineTo(x2, y2);
			this.context.stroke();
		},
	},
	mounted() {
		this.load();
	},
};
</script>

<style>
#app-container {
	height: 100%;
	width: 100%;
}

#bg-image {
	background-image: url("../../public/images/Dofus_CUKh7aUBMn.jpg");
	height: 100%;
	width: 100%;

	position: fixed;
	left: 0;
	right: 0;
	top: 10px;

	-webkit-filter: blur(3px);
	-moz-filter: blut(3px);
	-o-filter: blur(3px);
	-ms-filter: blur(3px);
	filter: blur(3px);
	z-index: 3;
}

#content {
	width: 100%;
	height: 100%;
	position: fixed;
	left: 0;
	right: 0;
	z-index: 5;
}

#isocanvas {
	width: 100%;
	height: 100%;
	position: fixed;
	z-index: 5;
}

.HUD {
	position: fixed;
	height: 100%;
	width: 100%;
	z-index: 5;
}

.HUD-left {
	position: fixed;
	height: 100%;
	width: 18%;
	background-color: rgba(0, 0, 0, 0.5);
}

.HUD-right {
	position: fixed;
	height: 100%;
	width: 18%;
	background-color: rgba(0, 0, 0, 0.5);
	right: 0;
}
</style>
