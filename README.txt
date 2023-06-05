This README file contains necessary changes to be made for the Clusterer.py file in regressinonly/functions/Clusterer.py to enable the "take_log" condition and normalization. The changes follow by the sequence of the lines. 

1. I added an instance variable called do_normalization takes a boolean that is initialized as "False" from the "normalization" parameter of the constructor of the Strawman_Clusterer.

2. In run_segmentation_clusterer() function, I added "if do_normalization: self.do_normalization()" condition after "self.apply_sampling_function" where "do_normalization" is a function that executes the normalization process, which will be implemented later (got an error).

3. In "get_cluster_sum()" funtion, I deleted 

    if (self_takelog):
        self.cluster_sum = np.log10(self.cluster_sum)
        
    because logging process would not take place in this part of the clusterer.
    
4. In "get_segmented_clusterer_sum()" function, I deleted 

    if (self.take_log):
            self.segmented_cluster_sum = np.log10(self.segmented_cluster_sum)
            self.cluster_sum = np.log10(self.cluster_sum)
            
    for the same reason as (3). Also, the initial way of np.logging the segmented_cluster_sum would not work because it is in a shape that cannot be normally
    computed by np.log10. It is a swapped axis 2 dimensional matrix.
    
5. No changes for get_genP() function. np.log10 for genP should be executed here since genP data do not undergo any other process since this function.

6. This is the whole implementation of "apply_sampling_fraction":

    def apply_sampling_fraction(self):

        self.cluster_sum = self.cluster_sum/self.sampling_fraction
        
        if self.take_log:
            self.cluster_sum = np.log10(self.cluster_sum)

        if self.segmented_cluster_sum_exist():
            self.segmented_cluster_sum = self.segmented_cluster_sum/self.sampling_fraction
            
            if self.take_log:
                for i in range(len(self.segmented_cluster_sum)):
                    for j in range(len(self.segmented_cluster_sum[i])):
                        if self.segmented_cluster_sum[i][j] == 0.0:
                            self.segmented_cluster_sum[i][j] = 1.0
                    self.segmented_cluster_sum[i] = np.log10(self.segmented_cluster_sum[i])
                

        print(f"Applied Sampling Fraction of {self.sampling_fraction} to Cluster Sums")
        
        This function is the last function that make changes to self.cluster_sum and self.segmented_cluster_sum. In other words, in the previous functions,
        those two instance variables are not in the final format. Therefore, when I apply np.log to cluster_sum and segmented_cluster_sum, the data points for 
        these variables do not correspond to its non_log conjugates. For instance, np.log(max(cluster_sum)) with no_log was not equal to max(cluster) with log.
        It is because in "sampling_fraction()," it mutates the data by dividing "cluster_sum" and "cluster_segmented_sum" by the "sampling_fraction." Therefore, 
        I had to apply log to the sums in "apply_sampling_function()."
        
        for cluster_sum, np.log(self.cluster_sum) will work because it is just a one dimensional np.array. For self.segmented_cluster_sum, first condition
        self.segmented_cluster_sum_exist() should be checked because this function runs for both non-segmented and segmented. Also, the major proportion of the
        sums are 0, and when np.log of 0 is taken, it yields an error because it results in -inf. Therefore, we have to iterate through every single points of
        the cluster_segmented_sum. Therefore, make a double for loops with i and j for indices, and make a condition that if the data point is 0, make it as 1,
        so it will compute to 0 when logged. Then, self.segmented_cluster_sum[i] = np.log(selg.segmented_cluster_sum[i]) is taken to compute the log's of a
        line. 
        
        