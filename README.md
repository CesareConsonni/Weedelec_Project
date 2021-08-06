# Weedelec_Project
Apply segmentation in order to distinguish crops from weeds

## Preliminary considerations
We chose to focus on the Weedelec dataset, because it seemed the clearest and most resoluted
one. In particular we started from Weedelec Mais, because it was the dataset with the most
detailed masks, and then we considered also Weedelec Haricot.

## First approach: basic encoder-decoder
We started with the net for multiclass segmentation proposed during the lecture, composed of
pretrained VGG as encoder and simple upsampling without skip connections. The network was
quite accurate in correctly classifying crop (mais only) and weed, but the details on the contour of
the segmentation was really poor (blob-like segmentation). We tried to change the encoder with a
more performat one (NASNet), but the results were similar.

![image](https://user-images.githubusercontent.com/37812489/128539655-57a9ffef-750e-442a-8277-e7d7dadff6e1.png)

## Second approach: U-net

We switched to a basic U-net, in order to exploit skip connections and achieve sharper details in
the contour of segmentation. We first just downsampled the full images to 512x512 and the
segmentation was immediately better, reaching a validation meanIoU around 0,50. Then we tried
with downsampling to 1024x1024: the segmentation was better again (around 0,58 meanIoU), but
slightly slower in training.
In order to increase the resolution of the segmentation, without exceeding the constraints on our
local GPU memory, we decided to use the tiling approach: downsampling less the original image
(keeping the aspect ratio) and cutting it into 6 square tiles (512x512). We chose to export the
tiles on disk in order to decrease the training time and increase the shuffling of the dataset. We
reached a validation meanIoU above 0,75. We tried to increase the size of the tiles to 800x800
(thus reducing downsampling factor), but without achieving a significant improvement compared
to the increase effort in training.
We were satisfied enough with the results, so we decided to add Haricot (still from Weedelec) in
order to challange ourselves on the complete dataset of a team. We saw that keeping the crop
classes (Mais and Haricot) separated brought better results in terms of accuracy of the
segmentation, so we modified the network to classsify 4 classes (background, weed, mais,
haricot). We kept the same approach (little downsampling, tiling in six 512x512 tiles exported on
disk), but we gave a different colour to haricot in the masks. Finally we shuffled all the tiles (from
both crops together) and trained the whole network. During classification of a new sample, the
crop classes (mais and haricot) are merged in output.
We trained the network in two steps, lowering the learning rate when we approached a plateau.
Below you find the training and validation meanIoU. In order to achieve better generalizaition we
used data augmentation during training.

![image](https://user-images.githubusercontent.com/37812489/128539745-e7ed9be7-4309-4c6b-9954-e646c956199f.png)

![image](https://user-images.githubusercontent.com/37812489/128539771-fa6ebf52-12ff-471e-a759-8b5abf5ad7a9.png)

On a sample image of the validation set we obtained this result (from left: original image, our
prediction and the true mask):

![image](https://user-images.githubusercontent.com/37812489/128539800-0fac4048-f886-496f-bfe8-1628471e6528.png)

## Conclusions
We finally classified the images from the test set, getting the following results:

![image](https://user-images.githubusercontent.com/37812489/128539855-13e09e87-c2fe-495f-ae56-5f69285b8170.png)

* We didn't used overlapping tiles, as the results didn't suffered of problems on the boundary
of the tiles (as you can see from the above image).
* We tried further to increase the number of layers of the network and adding more filters,
without significant improvement on the validation meanIoU.
* We tested the network also on the BipBip Haricot dataset (which was not used in training),
showing fairly acceptable generalization performance (as you can see from the image
below).

![image](https://user-images.githubusercontent.com/37812489/128539958-5a6a8ed3-5d43-4c90-bfd3-17e1137f8c19.png)



