<script>
    import electricData1 from './electric_data_window.json';
    import PlotElectric1 from './PlotElectric.svelte';
    import PlotElectric2 from './PlotElectricIFFT.svelte';

    import fft from 'fft-js';

    import {  writable } from 'svelte/store';
    import PlotElectricFft from './PlotElectricFFT.svelte';

    
    const voltages = electricData1.map(data => data.Voltage);
    const time = electricData1.map(data => data.Date_time);

    const n = voltages.length;
    const FFTresult = fft.fft(voltages);

    const frequencies = [];
    for (let i = 0; i < n; i ++){
        frequencies.push(i * (1/n));
    }


    const FFTConjugate = FFTresult.map(d => ({ real: d[0], imag: -d[1] }));

    const PSD = FFTresult.map((num, index) => ({
        real: num[0] * FFTConjugate[index].real - num[1] * FFTConjugate[index].imag,
        imag: num[0] * FFTConjugate[index].imag + num[1] * FFTConjugate[index].real
    }));

    const PSD_real = PSD.map(d => d.real/n)

    let threshold_input = 500;
    let threshold = writable();
    
    let dataStoreFFTMagnitudes = writable([]);
    let dataStoreIFFT = writable([]);


    $: {
        threshold.set(threshold_input);
        const FFTresult_thresholded = PSD_real.map((power, index) => power > threshold_input ? FFTresult[index] : [0, 0])
        const IFFTresult = fft.ifft(FFTresult_thresholded);

        dataStoreFFTMagnitudes.set(frequencies.slice(0, PSD_real.length / 2).map((t, index) => ({ frequency: t, magnitude: 2*PSD_real[index] })));
        dataStoreIFFT.set(time.map((t,index) => ({Date_time: t, Voltage: IFFTresult[index][0]})))
    }


</script>



<main>
    <PlotElectric1 {electricData1} />
    <PlotElectricFft {dataStoreFFTMagnitudes} {threshold}/>
    <PlotElectric2 {dataStoreIFFT} />
    <label>
        Threshold Slider:
        <input type="range" min="0" max="{2500}" step="1" bind:value={threshold_input} />
        {threshold_input}
    </label>
</main>