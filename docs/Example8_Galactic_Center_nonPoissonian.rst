
Example 8: Application of NPTFit to the Galactic Center Excess
==============================================================

It was found in Example 3 that a non-zero value of the GCE template is
preferred by a fit in the galactic center. In this example we will test
the point source interpretation of this excess by including, in addition
to the Poissonian templates considered, non-Poissonian point source
templates of various morphologies.

Here we simply perform the run, a detailed analysis of the results can
be found in the next example. A Python script version of this notebook
can be found as ``Example8_Galactic_Center_nonPoissonian.py``, which can
be run faster on multiple processors with MPI (see example in
``Example8_Galactic_Center_Batch.batch``.

**NB:** Even with ``nlive=100``, this notebook can take up to an hour to
complete. This highlights that for realistic non-Poissonian runs,
running on multiple cores becomes necessary. We show an explicit
application of this in Example 10.

**NB:** This example makes use of the Fermi Data, which needs to already
be installed. See Example 1 for details.

.. code:: ipython3

    # Import relevant modules
    
    %matplotlib inline
    %load_ext autoreload
    %autoreload 2
    
    import numpy as np
    
    from NPTFit import nptfit # module for performing scan
    from NPTFit import create_mask as cm # module for creating the mask
    from NPTFit import dnds_analysis # module for analysing the output
    from NPTFit import psf_correction as pc # module for determining the PSF correction

Step 1: Set up the Scan
-----------------------

We first need to 1. Set up an instance of ``NPTF`` from ``npfit.py`` 2.
Load in the data and exposure maps 3. Set up and load the mask used for
the scan 4. Load in the spatial templates

These are done identically to Example 3, and we refer to that notebook
for details.

.. code:: ipython3

    n = nptfit.NPTF(tag='GCE_Example')

.. code:: ipython3

    fermi_data = np.load('fermi_data/fermidata_counts.npy').astype(np.int32)
    fermi_exposure = np.load('fermi_data/fermidata_exposure.npy')
    n.load_data(fermi_data, fermi_exposure)

.. code:: ipython3

    pscmask=np.array(np.load('fermi_data/fermidata_pscmask.npy'), dtype=bool)
    analysis_mask = cm.make_mask_total(band_mask = True, band_mask_range = 2,
                                       mask_ring = True, inner = 0, outer = 30,
                                       custom_mask = pscmask)
    n.load_mask(analysis_mask)

.. code:: ipython3

    dif = np.load('fermi_data/template_dif.npy')
    iso = np.load('fermi_data/template_iso.npy')
    bub = np.load('fermi_data/template_bub.npy')
    gce = np.load('fermi_data/template_gce.npy')
    dsk = np.load('fermi_data/template_dsk.npy')
    
    n.add_template(dif, 'dif')
    n.add_template(iso, 'iso')
    n.add_template(bub, 'bub')
    n.add_template(gce, 'gce')
    n.add_template(dsk, 'dsk')
    
    # Remove the exposure correction for PS templates
    rescale = fermi_exposure/np.mean(fermi_exposure)
    n.add_template(gce/rescale, 'gce_np', units='PS')
    n.add_template(dsk/rescale, 'dsk_np', units='PS')

Step 2: Add Models
------------------

.. code:: ipython3

    n.add_poiss_model('dif', '$A_\mathrm{dif}$', fixed=True, fixed_norm=12.85)
    n.add_poiss_model('iso', '$A_\mathrm{iso}$', [0,2], False)
    n.add_poiss_model('gce', '$A_\mathrm{gce}$', [0,2], False)
    n.add_poiss_model('bub', '$A_\mathrm{bub}$', [0,2], False)

This time we add a non-Poissonian template correlated with the Galactic
Center Excess and also one spatially distributed as a thin disk. The
latter is designed to account for the unresolved point sources
attributed to the disk of the Milky Way (known sources in the 3FGL are
masked).

.. code:: ipython3

    n.add_non_poiss_model('gce_np',
                          ['$A_\mathrm{gce}^\mathrm{ps}$','$n_1^\mathrm{gce}$','$n_2^\mathrm{gce}$','$S_b^{(1), \mathrm{gce}}$'],
                          [[-6,1],[2.05,30],[-2,1.95],[0.05,40]],
                          [True,False,False,False])
    n.add_non_poiss_model('dsk_np',
                          ['$A_\mathrm{dsk}^\mathrm{ps}$','$n_1^\mathrm{dsk}$','$n_2^\mathrm{dsk}$','$S_b^{(1), \mathrm{dsk}}$'],
                          [[-6,1],[2.05,30],[-2,1.95],[0.05,40]],
                          [True,False,False,False])

Step 3: Configure Scan with PSF correction
------------------------------------------

.. code:: ipython3

    pc_inst = pc.PSFCorrection(psf_sigma_deg=0.1812)
    f_ary, df_rho_div_f_ary = pc_inst.f_ary, pc_inst.df_rho_div_f_ary


.. parsed-literal::

    Loading the psf correction from: /zfs/nrodd/NPTFRemakeExamples/psf_dir/gauss_128_0.181_10_50000_1000_0.01.npy


.. code:: ipython3

    n.configure_for_scan(f_ary, df_rho_div_f_ary, nexp=1)


.. parsed-literal::

    The number of parameters to be fit is 11


Step 4: Perform the Scan
------------------------

As noted above, we take a small value of ``nlive`` simply to ensure the
run finishes in a reasonable time on a single core.

.. code:: ipython3

    n.perform_scan(nlive=100)

This can take **up to an hour to run**. The output of this run will be
analyzed in detail in the next example.

.. code:: ipython3

    from IPython.display import Image
    Image(url = "https://imgs.xkcd.com/comics/compiling.png")




.. raw:: html

    <img src="https://imgs.xkcd.com/comics/compiling.png"/>


