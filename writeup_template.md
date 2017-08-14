## Project: Perception Pick & Place
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify). 
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  See the example `output.yaml` for details on what the output should look like.  
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  

#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.
Here is an example of how to include an image in your writeup.

![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)




Here's | A | Snappy | Table
--- | --- | --- | ---
1 | `highlight` | **bold** | 7.41
2 | a | b | c
3 | *italic* | text | 403
4 | 2 | 3 | abcd


### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

And here's another image! 
![demo-2](https://user-images.githubusercontent.com/20687560/28748286-9f65680e-7468-11e7-83dc-f1a32380b89c.png)

Spend some time at the end to discuss your code, what techniques you used, what worked and why, where the implementation might fail and how you might improve it if you were going to pursue this project further.  

## 1. Extract features and train an SVM model on new objects

Use the model that has been created in practice three. The diagram below model accuracy

![model_2](https://github.com/nnresearcher/RoboND-Perception-Project/blob/master/pr2_robot/pic/figure_1.png)

## 2. Write a ROS node and subscribe to `/pr2/world/points` topic

  We write some node as these:
`
    rospy.init_node('clustering', anonymous=True)
    
    pcl_sub = rospy.Subscriber("/pr2/world/points", pc2.PointCloud2, pcl_callback, queue_size=1)
    
    pcl_objects_pub = rospy.Publisher("/pcl_objects", PointCloud2, queue_size=1)
    
    pcl_table_pub = rospy.Publisher("/pcl_table", PointCloud2, queue_size=1)
    
    pcl_cluster_pub = rospy.Publisher("pcl_cluster", PointCloud2, queue_size=1)
    
    object_markers_pub = rospy.Publisher("/object_markers", Marker, queue_size=1)
    
    detected_objects_pub = rospy.Publisher("/detected_objects", DetectedObjectsArray, queue_size=1)
    
    base_controller_pub = rospy.Publisher("/pr2/world_joint_controller/command", Float64, queue_size=1)
    
`

The ROS node `pcl_sub` is subscribe to `/pr2/world/points` topic.

## 3. Use filtering and RANSAC plane fitting to isolate the objects 

  Just as we do in exercise 2, we creat 'Voxel Grid Downsampling'、'PassThrough Filter'、'Previous filters'、'RANSAC Plane Segmentation' to isolate the objects.

  The code is too long is not listed.

## 4. Apply Euclidean clustering to create separate clusters
   
    white_cloud = XYZRGB_to_XYZ(cloud_objects)# Apply function to convert XYZRGB to XYZ
    tree = white_cloud.make_kdtree()
    ec = white_cloud.make_EuclideanClusterExtraction()
    ec.set_ClusterTolerance(0.01)
    ec.set_MinClusterSize(20)
    ec.set_MaxClusterSize(3000)
    ec.set_SearchMethod(tree)
    cluster_indices = ec.Extract()
    
    
## 5. Perform object recognition on these objects and assign them labels (markers in RViz).

  We can see in the cognitive result of three scenarios
  
  In the first scenarios
  
  ![test1.world](https://github.com/nnresearcher/RoboND-Perception-Project/blob/master/pr2_robot/pic/1.png)
  
  My code recognize all objects to success in test1.world
  
  In the second scenarios
  
  ![test2.world](https://github.com/nnresearcher/RoboND-Perception-Project/blob/master/pr2_robot/pic/2.png)
  
  My code recognize all objects to success in test2.world
  
  In the third scenarios
  
  ![test3.world](https://github.com/nnresearcher/RoboND-Perception-Project/blob/master/pr2_robot/pic/3.png)
  
  My code recognize 7/8 objects to success in test3.world
  
## 6. Calculate the centroid

        points_arr = ros_to_pcl(object.cloud).to_array()
        object_centroid = np.mean(points_arr, axis=0)[:3]
        centroids.append(object_centroid)
        PICK_POSE.position.x = float(object_centroid[0])
        PICK_POSE.position.y = float(object_centroid[1])
        PICK_POSE.position.z = float(object_centroid[2])
        
  PICK_POSE.position.x 、PICK_POSE.position.y and PICK_POSE.position.z is object centroid
  
## 7. Create ROS messages containing the details

    TEST_SCENE_NUM  = Int32()
    TEST_SCENE_NUM.data = 3
    PICK_POSE = Pose()
    PLACE_POSE = Pose()
    WHICH_ARM = String()
    for i in range(len(dropbox_list)):
        for k  in range(len(object_list_param)):
            if object.label == object_list_param[k]['name']:
                if object_list_param[k]['group'] == dropbox_list[i]['group']:
                    PLACE_POSE.position.x = dropbox_list[i]['position'][0]
                    PLACE_POSE.position.y = dropbox_list[i]['position'][1]
                    PLACE_POSE.position.z = dropbox_list[i]['position'][2]
                    WHICH_ARM.data = str(dropbox_list[i]['name'])
    yaml_dict = make_yaml_dict(TEST_SCENE_NUM, WHICH_ARM, OBJECT_NAME, PICK_POSE, PLACE_POSE)
## 8. Write these messages

  Using the code 
  
      send_to_yaml('output_list_3.yaml', dict_list)
   
  I save the .yaml in https://github.com/nnresearcher/RoboND-Perception-Project
  
## That is all 
