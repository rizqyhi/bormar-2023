<script lang="ts">
	export let gpxFile: string;

	import type L from 'leaflet';
	import { onMount } from 'svelte';

	let L = null;
	let map = null;
	let gpxLayer = null;
	let markers = [];

	onMount(async () => {
		L = await import('leaflet-gpx');
		map = L.map('map').setView([-7.6079, 110.2038], 16);
		L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
			maxZoom: 16,
			minZoom: 13,
			attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
		}).addTo(map);
	});

	$: if (L && map) {
		if (gpxLayer) {
			map.removeLayer(gpxLayer);
		}

		if (markers) {
			markers.forEach((marker) => marker.removeFrom(map));
		}

		function flagIcon() {
			return L.divIcon({
				html: `<div class="waypoint-marker__flag">🚩</div>`,
				iconSize: [48, 48],
				className: `waypoint-marker`
			});
		}

		function hiddenIcon() {
			return L.divIcon({
				iconSize: [32, 32],
				className: `hidden`
			});
		}

		function combinedIcon(latlng: number[], icon: string) {
			const markerTypes = icon.split('_');
			const kmMarker = icon.match(/KM(\d+)/);
			let iconHtml = '';

			if (kmMarker) {
				iconHtml += `<div class="waypoint-marker__km">${kmMarker[1]}</div>`;
			}

			if (markerTypes.includes('WS')) {
				iconHtml += `<div class="waypoint-marker__ws">💧</div>`;
			}

			if (markerTypes.includes('MEDIC')) {
				iconHtml += `<div class="waypoint-marker__medic">🏥</div>`;
			}

			if (markerTypes.includes('AMBULANCE')) {
				iconHtml += `<div class="waypoint-marker__ambulance">🚑</div>`;
			}

			if (markerTypes.includes('FRUIT')) {
				iconHtml += `<div class="waypoint-marker__ambulance">🍉</div>`;
			}

			if (markerTypes.includes('SHOWER')) {
				iconHtml += `<div class="waypoint-marker__ambulance">🚿</div>`;
			}

			if (markerTypes.includes('SPONGE')) {
				iconHtml += `<div class="waypoint-marker__ambulance">🧽</div>`;
			}

			const iconLayer = L.divIcon({
				html: iconHtml,
				iconSize: [32 * markerTypes.length + 16, 48],
				className: `waypoint-marker`
			});

			return L.marker(latlng, { icon: iconLayer });
		}

		gpxLayer = new L.GPX(`/${gpxFile}`, {
			async: true,
			polyline_options: {
				color: '#ef4444',
				opacity: 0.85,
				weight: 5,
				lineCap: 'round'
			},
			marker_options: {
				startIcon: flagIcon(),
				endIcon: flagIcon(),
				pointMatchers: [
					{
						regex: /.+/,
						icon: hiddenIcon()
					}
				]
			}
		})
			.on('loaded', function (e) {
				map.fitBounds(e.target.getBounds());
			})
			.on('addpoint', function (e) {
				if (e.point_type !== 'waypoint') return;

				markers.push(
					combinedIcon([e.point._latlng.lat, e.point._latlng.lng], e.point.options.title).addTo(map)
				);
			})
			.addTo(map);
	}
</script>

<div id="map" style="height: calc(100vh - 64px)" />
