# **Hadoop Max-Min Temperature MapReduce Program – Documentation**

### **1. Prerequisites**

* Hadoop installed and configured on Ubuntu.
* HDFS running (`start-dfs.sh` and `start-yarn.sh`).
* Java installed (Java 11 recommended).
* Environment variables set:

```bash
echo $HADOOP_HOME
echo $PATH
```

* Input CSV file `temperature.csv` with format:

```
year,temperature
2000,45
2001,54
2002,34
2003,54
```

---

### **2. Create Project Folder**

```bash
mkdir -p ~/hadoop_programs/maxmintemp
cd ~/hadoop_programs/maxmintemp
```

---

### **3. Create Java Program**

Create the Java file `MaxMinTemperature.java`:

```bash
nano MaxMinTemperature.java
```

**Paste this code inside:**

```java
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MaxMinTemperature {

    // Mapper Class
    public static class TempMapper extends Mapper<Object, Text, Text, IntWritable> {
        private Text type = new Text("temp"); // single key to send all data to one reducer

        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString().trim();
            if (line.startsWith("year") || line.isEmpty()) return; // skip header
            String[] parts = line.split(",");
            int temp = Integer.parseInt(parts[1].trim());
            context.write(type, new IntWritable(temp));
        }
    }

    // Reducer Class
    public static class TempReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        public void reduce(Text key, Iterable<IntWritable> values, Context context)
                throws IOException, InterruptedException {
            int maxTemp = Integer.MIN_VALUE;
            int minTemp = Integer.MAX_VALUE;

            // find max and min temperatures
            for (IntWritable val : values) {
                int t = val.get();
                if (t > maxTemp) maxTemp = t;
                if (t < minTemp) minTemp = t;
            }

            context.write(new Text("Maximum Temperature:"), new IntWritable(maxTemp));
            context.write(new Text("Minimum Temperature:"), new IntWritable(minTemp));
        }
    }

    // Main Method
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("Usage: MaxMinTemperature <input path> <output path>");
            System.exit(-1);
        }

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "Max and Min Temperature");
        job.setJarByClass(MaxMinTemperature.class);
        job.setMapperClass(TempMapper.class);
        job.setReducerClass(TempReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

### **4. Compile Java Program**

```bash
# Make sure class name matches file name
javac -source 11 -target 11 -classpath `hadoop classpath` -d . MaxMinTemperature.java
```

**Note:** If you see a warning about system modules, it can usually be ignored.

---

### **5. Create JAR File**

```bash
jar -cvf maxmintemp.jar MaxMinTemperature*.class
```

---

### **6. Upload Input File to HDFS**

```bash
# Create input directory
hdfs dfs -mkdir -p /input

# Upload local file
hdfs dfs -put ~/hadoop_programs/maxmintemp/temperature.csv /input

# Verify
hdfs dfs -ls /input
```

Output should show:

```
-rw-r--r--   1 grace supergroup          20 2025-11-08 14:00 /input/temperature.csv
```

---

### **7. Run Hadoop Job**

```bash
# Remove previous output folder if exists
hdfs dfs -rm -r /output

# Run the MapReduce job
hadoop jar ~/hadoop_programs/maxmintemp/maxmintemp.jar MaxMinTemperature /input /output
```

* If successful, Hadoop will print `Job Finished successfully` in the console.

---

### **8. View Output**

```bash
hdfs dfs -ls /output
hdfs dfs -cat /output/part-r-00000
```

Expected output:

```
Maximum Temperature: 54
Minimum Temperature: 34
```

---

### **9. Optional: Copy Output to Local Machine**

```bash
hdfs dfs -get /output ~/hadoop_programs/maxmintemp/local_output
```

Then you can view the result locally:

```bash
cat ~/hadoop_programs/maxmintemp/local_output/part-r-00000
```

---

### **10. Cleanup**

```bash
# Remove input and output from HDFS
hdfs dfs -rm -r /input
hdfs dfs -rm -r /output
```

---


---

If you want, I can also create an **enhanced version** that outputs **the year along with the max and min temperatures** — so you’ll see **2001,54** instead of just 54. This is often more useful for reports.

Do you want me to create that version?
