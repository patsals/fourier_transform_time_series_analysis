<script>
    import electricData1 from './electric_data_window.json';
    import electricData2 from './electric_data_next_window.json';
    import PlotElectric1 from './PlotElectric.svelte';
    import PlotElectric2 from './PlotElectricIFFT.svelte';
    import PlotElectric3 from './PlotElectricIFFTnext.svelte';


    import fft from 'fft-js';

    import {  writable } from 'svelte/store';
    import PlotElectricFft from './PlotElectricFFT.svelte';

    
    const voltages = electricData1.map(data => data.Voltage);
    const time = electricData1.map(data => data.Date_time);
    const time_next = electricData2.map(data => data.Date_time);

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

    let threshold_input = 1000;
    let threshold = writable();
    
    let dataStoreFFTMagnitudes = writable([]);
    let dataStoreIFFT = writable([]);
    let dataStoreIFFTnext = writable([]);

    $: {
        threshold.set(threshold_input);
        const FFTresult_thresholded = PSD_real.map((power, index) => power > threshold_input ? FFTresult[index] : [0, 0])
        const IFFTresult = fft.ifft(FFTresult_thresholded);

        dataStoreFFTMagnitudes.set(frequencies.slice(0, PSD_real.length / 2).map((t, index) => ({ frequency: t, magnitude: 2*PSD_real[index] })));
        dataStoreIFFT.set(time.map((t,index) => ({Date_time: t, Voltage: IFFTresult[index][0]})))
        dataStoreIFFTnext.set(time_next.map((t,index) => ({Date_time: t, Voltage: IFFTresult[index][0]})))
    }


</script>



<main>
    <h2>Let's consider real-world data</h2>

    <h3>Voltage readings sampled from a US household in February of 2023:</h3>
    <PlotElectric1 {electricData1} />
    <h3>If we compute the power spectrum of this data, which is obtained by taking the squared magnitude of the Fourier Transform, we can get the following:</h3>
    <PlotElectricFft {dataStoreFFTMagnitudes} {threshold}/>
    <h4>
        Threshold Slider:
        <input type="range" min="0" max="10000" step="1" bind:value={threshold_input} style="width: 300px;" />
        {threshold_input}
    </h4>
    <h3>Now we can apply thresholding to the different magnitudes, and perform the Inverse Fourier Transform to obtain a denoisified version of the data:</h3>
    <PlotElectric2 {dataStoreIFFT} {electricData1}/>
    <h3>Which we can then use to model next week's voltage values!</h3>
    <PlotElectric3 {dataStoreIFFTnext} {electricData2}/>
</main>