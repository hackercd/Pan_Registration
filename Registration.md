# Pan_Registration
Scripts and code for rat image registration in Pan et al., 2022. 

The following code was used to register a representative image from this study, "062419.nii", to at atlas image obtained from Johnson et al., 2012, "p08_average_gre.nii" (Johnson GA, Calabrese E, Badea A, Paxinos G, Watson C. A multidimensional magnetic resonance histology atlas of the Wistar rat brain. Neuroimage. 2012 Sep;62(3):1848-56.) A combination of Matlab, Python, and in-house tools ("4dfp" image processing tools, https://4dfp.readthedocs.io/en/latest/) were used. 


%% Crop image (Matlab code)
filename='062419.nii';
output_path='062419_crop.nii';
info=niftiinfo(filename);
V = niftiread(filename);
info.PixelDimensions=[0.0147 0.0147 0.0147];
info.ImageSize=info.ImageSize(1:3);
Vcrop=V(400:1700,400:1700,:);
info.SliceEnd=size(Vcrop,3)-1;
info.ImageSize=size(Vcrop);
niftiwrite(Vcrop,output_path,info)

## Resample image (Python code)
import nibabel
import nibabel.processing
input_path = r'062419_crop.nii'
output_path = r'062419_0.04.nii'
input_img = nibabel.load(input_path)
voxel_size = [0.04 , 0.04 , 0.04]
resampled_img = nibabel.processing.resample_to_output(input_img, voxel_size)
nibabel.save(resampled_img, output_path)

%% Crop Atlas (Matlab code)
V = niftiread('p08_average_gre.nii');
info = niftiinfo('p08_average_gre.nii');
% size is [920   560   520]
Vcrop = single(zeros(920,920,920))
Vcrop(:,  920/2-560/2+1:920/2+560/2  , 920/2-520/2+1:920/2+520/2)=V;
Vcrop=Vcrop(1:2:end,1:2:end,1:2:end);
info.PixelDimensions=info.PixelDimensions*2;
info.ImageSize=size(Vcrop);
niftiwrite(Vcrop,'C:\Rat\p08_sq50um.nii',info)

#ORIENT ATLAS
t4img_4dfp /data/nil-bluearc/raichle/lin64-tools/zrot90_t4 p08_sq50um p08_sq50um_zrot90 -Op08_sq50um
flip_4dfp -y p08_sq50um_zrot90
set atlimg = p08_sq50um_zrot90_flipy

#ORIENT CT
set CTimg = 062419_0.04
t4img_4dfp /data/nil-bluearc/raichle/lin64-tools/xrot90_t4 062419_0.04 062419_0.04_xrot90 -O062419_0.04
flip_4dfp -z 062419_0.04_xrot90

#PERFORM IMAGE REGISTRATION
set t4file = ${CTimg}_to_${atlimg}_t4
imgreg_4dfp $atlimg none $CTimg none $t4file 519
t4img_4dfp $t4file $CTimg ${CTimg}_on_${atlimg}

#"${CTimg}_on_${atlimg}" yields the filename of the image registered onto the atlas
