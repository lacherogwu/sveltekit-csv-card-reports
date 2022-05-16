<script>
	// @ts-nocheck

	import Papa from 'papaparse';

	/**
	 *
	 * @param {File} file
	 * @returns {Promise<Array>}
	 */
	const parseFile = (file) =>
		new Promise((resolve, reject) =>
			Papa.parse(file, {
				header: false,
				complete({ data }) {
					resolve(data);
				}
			})
		);

	/** @type {FileList} */
	let files;

	const generateCsv = (items) => {
		const { name, account } = items[0];

		const csv = Papa.unparse(items);

		const csvData = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
		const csvURL = window.URL.createObjectURL(csvData);

		const tempLink = document.createElement('a');
		tempLink.href = csvURL;
		tempLink.setAttribute('download', `${name} ${account}.csv`);
		tempLink.click();
	};

	const handler = async () => {
		const [file] = files;
		const data = await parseFile(file);
		data.shift();
		data.shift();

		const mapData = (data) => {
			const _data = data.map((item) => item.trim());
			const [
				,
				date,
				receipt,
				description,
				name,
				account,
				amount,
				extendedDetails,
				appearOnStatementAs,
				address,
				cityState,
				zipCode,
				country,
				reference,
				category
			] = _data;

			return {
				account,
				date,
				name,
				description,
				amount,
				extendedDetails,
				appearOnStatementAs,
				address,
				cityState,
				zipCode,
				country,
				reference,
				category
			};
		};

		const list = data.reduce((acc, curr) => {
			const data = mapData(curr);
			const { account } = data;

			if (!acc[account]) acc[account] = [];

			acc[account].push(data);

			return acc;
		}, {});

		Object.values(list).forEach(generateCsv);
	};
</script>

<h1 class="text-center text-orange-500 text-7xl font-thin my-8">Download Reports</h1>

<section class="max-w-lg mx-auto border rounded shadow-lg p-8">
	<h1 class="text-gray-500 mb-2 text-xs">
		This is an example project, please don't pay attention to small details.
	</h1>

	<input type="file" accept=".csv" bind:files />

	<button
		class="px-4 py-2 rounded-full border transition-colors hover:bg-orange-500 border-orange-500 hover:text-white"
		on:click={handler}
	>
		Download
	</button>
</section>
