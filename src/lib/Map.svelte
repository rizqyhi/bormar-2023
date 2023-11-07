<script lang="ts">
	export let gpxFile: string;

	import { onMount } from 'svelte';
	import doctor from '$lib/icons/doctor.svg?raw';
	import glass from '$lib/icons/glass.svg?raw';
	import ambulance from '$lib/icons/ambulance.svg?raw';

	onMount(async () => {
		const L = await import('leaflet-gpx');
		const map = L.map('map').setView([-7.6079, 110.2038], 16);

		L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
			maxZoom: 16,
			minZoom: 13,
			attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
		}).addTo(map);

		function flagIcon() {
			return L.divIcon({
				html: `<div class="waypoint-marker__flag"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 -960 960 960" fill="currentColor"><path d="M360-720h80v-80h-80v80Zm160 0v-80h80v80h-80ZM360-400v-80h80v80h-80Zm320-160v-80h80v80h-80Zm0 160v-80h80v80h-80Zm-160 0v-80h80v80h-80Zm160-320v-80h80v80h-80Zm-240 80v-80h80v80h-80ZM200-160v-640h80v80h80v80h-80v80h80v80h-80v320h-80Zm400-320v-80h80v80h-80Zm-160 0v-80h80v80h-80Zm-80-80v-80h80v80h-80Zm160 0v-80h80v80h-80Zm80-80v-80h80v80h-80Z"/></svg></div>`,
				iconSize: [32, 32],
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
				iconHtml += `<div class="waypoint-marker__ws">${glass}</div>`;
			}

			if (markerTypes.includes('MEDIC')) {
				iconHtml += `<div class="waypoint-marker__medic">${doctor}</div>`;
			}

			if (markerTypes.includes('AMBULANCE')) {
				iconHtml += `<div class="waypoint-marker__ambulance">${ambulance}</div>`;
			}

			const iconLayer = L.divIcon({
				html: iconHtml,
				iconSize: [32 * markerTypes.length, 32],
				className: `waypoint-marker`
			});

			return L.marker(latlng, { icon: iconLayer });
		}

		new L.GPX(`/${gpxFile}`, {
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

				combinedIcon([e.point._latlng.lat, e.point._latlng.lng], e.point.options.title).addTo(map);
			})
			.addTo(map);
	});
</script>

<div id="map" style="height: calc(100vh - 64px)" />
