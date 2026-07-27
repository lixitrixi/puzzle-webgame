<script lang="ts">
	const columns = 10;
	const rows = 20;
	const cell = 28;
	const piece = [[1, 1, 1, 1]] as const;
	const blockColor = 0xeeeeee;

	let hoverColumn = 3;
	let hoverRow = 8;

	function getBoardFrame(width: number, height: number) {
		const boardWidth = columns * cell;
		const boardHeight = rows * cell;
		const boardX = Math.round((width - boardWidth) / 2);
		const boardY = Math.round((height - boardHeight) / 2);

		return { boardWidth, boardHeight, boardX, boardY };
	}

	function pixiStage(node: HTMLDivElement) {
		let cancelled = false;
		let dispose = () => {};

		void (async () => {
			const pixi = await import('pixi.js');
			if (cancelled) return;

			const application = new pixi.Application();
			await application.init({
				resizeTo: node,
				autoDensity: true,
				antialias: true,
				backgroundAlpha: 0,
				resolution: Math.max(1, window.devicePixelRatio || 1)
			});

			if (cancelled) {
				application.destroy({ removeView: true }, true);
				return;
			}

			node.appendChild(application.canvas);

			const scene = new pixi.Container();
			application.stage.addChild(scene);

			function render() {
				const { width, height } = application.screen;
				const { boardX, boardY } = getBoardFrame(width, height);

				scene.removeChildren().forEach((child) => child.destroy());

				const backdrop = new pixi.Graphics().rect(0, 0, width, height).fill({ color: 0x2563eb });
				const gridBase = new pixi.Graphics().setStrokeStyle({
					width: 1,
					color: blockColor,
					alignment: 0.5
				});
				const ghost = new pixi.Graphics().fill({ color: blockColor });

				for (let y = 0; y < rows; y += 1) {
					for (let x = 0; x < columns; x += 1) {
						gridBase.rect(boardX + x * cell, boardY + y * cell, cell, cell);
					}
				}
				gridBase.stroke();

				const ghostX = boardX + hoverColumn * cell;
				const ghostY = boardY + hoverRow * cell;

				for (let y = 0; y < piece.length; y += 1) {
					for (let x = 0; x < piece[y].length; x += 1) {
						if (!piece[y][x]) continue;
						ghost.rect(ghostX + x * cell + 0, ghostY + y * cell + 0, cell - 0, cell - 0);
					}
				}

				ghost.fill();
				scene.addChild(backdrop, gridBase, ghost);
			}

			function updateHover(event: PointerEvent) {
				const rect = node.getBoundingClientRect();
				const x = event.clientX - rect.left;
				const y = event.clientY - rect.top;
				const { boardX, boardY } = getBoardFrame(rect.width, rect.height);

				hoverColumn = Math.max(
					0,
					Math.min(columns - piece[0].length, Math.floor((x - boardX) / cell))
				);
				hoverRow = Math.max(0, Math.min(rows - piece.length, Math.floor((y - boardY) / cell)));
				render();
			}

			function clearHover() {
				hoverColumn = 3;
				hoverRow = 8;
				render();
			}

			const resizeObserver = new ResizeObserver(() => render());
			resizeObserver.observe(node);
			node.addEventListener('pointermove', updateHover);
			node.addEventListener('pointerleave', clearHover);
			render();

			dispose = () => {
				resizeObserver.disconnect();
				node.removeEventListener('pointermove', updateHover);
				node.removeEventListener('pointerleave', clearHover);
				application.destroy({ removeView: true }, true);
			};
		})();

		return () => {
			cancelled = true;
			dispose();
		};
	}
</script>

<div {@attach pixiStage} class="min-h-screen bg-blue-500"></div>
