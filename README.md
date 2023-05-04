Download Link: https://assignmentchef.com/product/solved-cs5477-lab-2-camera-calibration
<br>
In this assignment, you will implement Zhenyou Zhang’s camera calibration. The extrincs and intrincs of a camera are estimated from three images of a model plane. You will first estimate the five intrinsic parameters (focal length, principle point, skew) and six extrinsic parameters (three for rotation and three for translation) by a close-form solution. Then you will estimate five distortion parameters and also finetune all parameters by minimize the total reprojection error.

<h1>        1.0.2      Instructions</h1>

This workbook provides the instructions for the assignment, and facilitates the running of your code and visualization of the results. For each part of the assignment, you are required to <strong>complete the implementations of certain functions in the accompanying python file </strong>(lab2.py).

To facilitate implementation and grading, all your work is to be done in that file, and <strong>you only have to submit the .py file</strong>.

Please note the following:

<ol>

 <li>Fill in your name, email, and NUSNET ID at the top of the python file.</li>

 <li>The parts you need to implement are clearly marked with the following:</li>

</ol>

“‘

“”” YOUR CODE STARTS HERE “””

“”” YOUR CODE ENDS HERE “””

“‘

and you should write your code in between the above two lines.

<ol start="3">

 <li>Note that for each part, there may certain functions that are prohibited to be used. It is important <strong>NOT to use those prohibited functions </strong>(or other functions with similar functionality). If you are unsure whether a particular function is allowed, feel free to ask any of the TAs.</li>

</ol>




<h1>        1.1       Part 1: Load and Visualize Data</h1>

In this part, you will get yourself familiar with the data by visualizing it. The data includes three images of a planar checkerboard (CalibIm1-3.tif) and the correpsonding corner locations in each image (data1-3.txt). The 3D points of the model are stored in Model.txt. Note that only <em>X </em>and <em>Y </em>coordinates are provided because we assume that the model plane is on <em>Z </em>= 0. You can visualize the data with the provided code below.

<table>

 <tbody>

  <tr>

   <td width="251"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

<h1>        1.2      Part 2: Estimate the Intrinsic Parameters</h1>

In this part, you will estimate the the intrinsics, which inludes focal length, skew and principle point.You will firstly estimate the homography between each observed image and the 3D model. Note that you are allowed to use cv2.findHomography() here to since you already implemented it in lab1. Each view of the checkerboard gives us two constraints:

<strong>vb </strong>= <strong>0</strong>,

where <strong>v </strong>is 2 × 6 matrix made up of the homography terms. Given three observations, we can get :

<strong>Vb </strong>= <strong>0</strong>,

where <strong>V </strong>is a 6 × 6 matrix obtained from stacking all constraints together. The solution can be obtained by taking the right null-space of <strong>V</strong>, which is the right singular vector corresponding to the smallest singular value of <strong>V</strong>.

<h2>Implement the following function(s): cv2.calibrateCamera()</h2>

<ul>

 <li>You may use the following functions: cv2.findHomography(), np.linalg.svd()</li>

 <li>Prohibited Functions: cv2.calibrateCamera()</li>

</ul>

<h1>        1.3      Part 3: Estimate the Extrinsic Parameters</h1>

In this part, you will estimate the extrinsic parameters based on the intrinsic matrix <strong>A </strong>you obtained from Part 2. You can compute the rotation and translation according to:

<strong>r</strong>1 = <em>λ</em><strong>A</strong>−1<strong>h</strong>1<strong>r</strong>2 = <em>λ</em><strong>A</strong>−1<strong>h</strong>2<strong>r</strong>3 = <strong>r</strong>1 × <strong>r</strong>2<strong>t </strong>= <em>λ</em><strong>A</strong>−1<strong>h</strong>3.

<em>λ </em>= 1/k<strong>A</strong><sup>−1</sup><strong>h</strong><sub>1</sub>k = 1/k<strong>A</strong><sup>−1</sup><strong>h</strong><sub>2</sub>k, and <strong>h</strong><em><sub>i </sub></em>represents the <em>i<sup>th </sup></em>column of the homography <strong>H</strong>. Note that the rotation matrix <strong>R </strong>= [<strong>r</strong><sub>1</sub>, <strong>r</strong><sub>1</sub>, <strong>r</strong><sub>1</sub>] does not in general satisfy the properties of a rotation matrix. Hence, you will use the provided function convt2rotation() to estimate the best rotation matrix. The detail is given in the supplementary of the reference paper. • You may use the following functions:

np.linalg.svd(), np.linalg.inv(),np.linalg.norm(), convt2rotation

[3]: R_all, T_all, K = init_param(pts_model, pts_2d)

A = np.array([K[0], K[1], K[2], 0, K[3], K[4], 0, 0, 1]).reshape([3, 3]) img_all = [] for i in range(len(R_all)):

R = R_all[i] T = T_all[i] points_2d = pts_2d[i]

trans = np.array([R[:, 0], R[:, 1], T]).T points_rep = np.dot(A, np.dot(trans, pts_model_homo)) points_rep = points_rep[0:2] / points_rep[2:3] img = cv2.imread(‘./zhang_data/CalibIm{}.tif’.format(i + 1)) for j in range(points_rep.shape[1]):

cv2.circle(img, (np.int32(points_rep[0, j]), np.int32(points_rep[1,␣

<em>,</em><sub>→</sub>j])), 5, (0, 0, 255), 2) cv2.circle(img, (np.int32(points_2d[0, j]), np.int32(points_2d[1, j])),␣

<em>,</em><sub>→</sub>4, (255, 0, 0), 2) plt.figure() plt.imshow(img)

Up to now, you already get a rough estimation of the intrinsic and extrinsic parameters. You can check your results with the provided code, which visualizes the reprojections of the corner locations with the estimated parameters. You will find that the points that are far from the center of the image (the four corners of the checkerboard) are not as accurate as points at the center. This is because we did not consider the distortion parameters in this step.

<h1>        1.4      Part 4: Estimate All Parameters</h1>

In this part, you will estimate all parameters by minimize the total reprojection error:

<em>n  m </em>argmin ∑∑k<strong>x</strong><em><sub>ij </sub></em>−<em><sub>π</sub></em>(<strong>K</strong>, <strong>R</strong>, <strong>t</strong>, <strong>ˇ</strong>, <strong>X</strong><em><sub>j</sub></em>)k.

<strong>K</strong>,<strong>R</strong>,<strong>t</strong>,<strong>ˇ </strong><em>i</em>=1 <em>j</em>=1

<strong>K</strong>, <strong>R</strong>, <strong>t </strong>are the intrinsics and extrinsices, which are initialized with estimation from Part 3. <strong>ˇ </strong>represents the five distortion parameters and are initialized with zeros. <strong>X</strong><em><sub>j </sub></em>and <strong>x</strong><em><sub>ij </sub></em>represent the 3D model and the corresponding 2D observation.

Note that you will use the function least_squares() in scipy to minimize the reprojection error and find the optimal parameters. During the optimization process, the rotation matrix <strong>R </strong>should be represented by a 3-dimensional vector by using the provided function matrix2vector(). We provide the skeleton code of how to use the function least_squares() below.

The key step of the optimization is to define the error function error_fun(), where the first parameter param is the parameters you will optimize over. The param in this example includes: intrinsics (0-5), distortion (5-10), extrinsics (10-28). The extrincs consist of three pairs of rotation <strong>s </strong>and translation <strong>t </strong>because we have three views. The rotation <strong>s </strong>is the 3-dimensional vector representation, which you can convert back to a rotation matrix with provided function vector2matrix(). You will have to consider the distortion when computing the reprojection error. Let <strong>x </strong>= (<em>x</em>, <em>y</em>) be the normalized image coordinate, namely the points_ud_all in the code. The radial distortion is given by:

x<sub>r </sub>=  ,

where <em>r</em><sup>2 </sup>= <em>x</em><sup>2 </sup>+ <em>y</em><sup>2 </sup>and <em>κ</em><sub>1</sub>, <em>κ</em><sub>2</sub>, <em>κ</em><sub>5 </sub>are the radial distortion parameters. The tangential distortion is given by :

dx  ,

where <em>κ</em><sub>3</sub>, <em>κ</em><sub>4 </sub>are the tangential distortion parameters. FInally, the image coordinates after distortion is given by :

x<sub>d </sub>= x<sub>r </sub>+ dx.

The optimization converges when the error does not change too much. Note that you will decide the iter_num according to the error value by yourself. You can verify the optimal parameters by visualizing the points after distortion. The function visualize_distorted() is an example of how to visualize the the points after distortion in image. You will find that the points that are far from the center of the image is more accurate than the estimation from Part 3.