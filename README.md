# oop24
https://github.com/kdmitruk
https://github.com/lukaszkurantdev
https://github.com/michaldziuba03/java

Zdjęcia i wątki:
import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

public class ImageProcessor {
    private BufferedImage image;

    public void loadImage(String path) throws IOException {
        image = ImageIO.read(new File(path));
    }

    public void saveImage(String path) throws IOException {
        ImageIO.write(image, "png", new File(path));
    }

    public BufferedImage getImage() {
        return image;
    }

    public void setImage(BufferedImage image) {
        this.image = image;
    }

    public void increaseBrightness(int value) {
        for (int x = 0; x < image.getWidth(); x++) {
            for (int y = 0; y < image.getHeight(); y++) {
                int rgba = image.getRGB(x, y);
                int alpha = (rgba >> 24) & 0xff;
                int red = Math.min(255, ((rgba >> 16) & 0xff) + value);
                int green = Math.min(255, ((rgba >> 8) & 0xff) + value);
                int blue = Math.min(255, (rgba & 0xff) + value);

                int newRGBA = (alpha << 24) | (red << 16) | (green << 8) | blue;
                image.setRGB(x, y, newRGBA);
            }
        }
    }

    public void increaseBrightnessMultithreaded(int value) throws InterruptedException {
        int cores = Runtime.getRuntime().availableProcessors();
        Thread[] threads = new Thread[cores];
        int height = image.getHeight();
        int chunkSize = height / cores;

        for (int i = 0; i < cores; i++) {
            final int start = i * chunkSize;
            final int end = (i == cores - 1) ? height : (i + 1) * chunkSize;
            threads[i] = new Thread(() -> {
                for (int x = 0; x < image.getWidth(); x++) {
                    for (int y = start; y < end; y++) {
                        int rgba = image.getRGB(x, y);
                        int alpha = (rgba >> 24) & 0xff;
                        int red = Math.min(255, ((rgba >> 16) & 0xff) + value);
                        int green = Math.min(255, ((rgba >> 8) & 0xff) + value);
                        int blue = Math.min(255, (rgba & 0xff) + value);

                        int newRGBA = (alpha << 24) | (red << 16) | (green << 8) | blue;
                        image.setRGB(x, y, newRGBA);
                    }
                }
            });
            threads[i].start();
        }

        for (Thread thread : threads) {
            thread.join();
        }
    }

    public void increaseBrightnessThreadPool(int value) throws InterruptedException {
        int height = image.getHeight();
        ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

        for (int y = 0; y < height; y++) {
            final int row = y;
            executor.submit(() -> {
                for (int x = 0; x < image.getWidth(); x++) {
                    int rgba = image.getRGB(x, row);
                    int alpha = (rgba >> 24) & 0xff;
                    int red = Math.min(255, ((rgba >> 16) & 0xff) + value);
                    int green = Math.min(255, ((rgba >> 8) & 0xff) + value);
                    int blue = Math.min(255, (rgba & 0xff) + value);

                    int newRGBA = (alpha << 24) | (red << 16) | (green << 8) | blue;
                    image.setRGB(x, row, newRGBA);
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(Long.MAX_VALUE, TimeUnit.NANOSECONDS);
    }

    public int[] calculateHistogram(int channel) throws InterruptedException, ExecutionException {
        int[] histogram = new int[256];
        int height = image.getHeight();
        ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
        List<Future<int[]>> futures = new ArrayList<>();

        for (int y = 0; y < height; y++) {
            final int row = y;
            futures.add(executor.submit(() -> {
                int[] rowHistogram = new int[256];
                for (int x = 0; x < image.getWidth(); x++) {
                    int rgba = image.getRGB(x, row);
                    int value = 0;
                    if (channel == 0) {
                        value = (rgba >> 16) & 0xff; // Red channel
                    } else if (channel == 1) {
                        value = (rgba >> 8) & 0xff; // Green channel
                    } else if (channel == 2) {
                        value = rgba & 0xff; // Blue channel
                    }
                    rowHistogram[value]++;
                }
                return rowHistogram;
            }));
        }

        for (Future<int[]> future : futures) {
            int[] rowHistogram = future.get();
            for (int i = 0; i < histogram.length; i++) {
                histogram[i] += rowHistogram[i];
            }
        }

        executor.shutdown();
        executor.awaitTermination(Long.MAX_VALUE, TimeUnit.NANOSECONDS);

        return histogram;
    }

    public BufferedImage createHistogramImage(int[] histogram) {
        int width = 256;
        int height = 100;
        BufferedImage histogramImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);
        Graphics2D g = histogramImage.createGraphics();
        g.setColor(Color.WHITE);
        g.fillRect(0, 0, width, height);

        int max = 0;
        for (int value : histogram) {
            if (value > max) {
                max = value;
            }
        }

        g.setColor(Color.BLACK);
        for (int i = 0; i < histogram.length; i++) {
            int barHeight = (int) (((double) histogram[i] / max) * height);
            g.drawLine(i, height, i, height - barHeight);
        }

        g.dispose();
        return histogramImage;
    }
}

public class Main {
    public static void main(String[] args) {
        try {
            ImageProcessor processor = new ImageProcessor();

            // Wczytywanie obrazu
            processor.loadImage("input.png");

            // Zwiększenie jasności obrazu
            processor.increaseBrightness(50);
            processor.saveImage("output_brightness.png");

            // Zwiększenie jasności obrazu wielowątkowo
            processor.loadImage("input.png");
            processor.increaseBrightnessMultithreaded(50);
            processor.saveImage("output_brightness_multithreaded.png");

            // Zwiększenie jasności obrazu z użyciem puli wątków
            processor.loadImage("input.png");
            processor.increaseBrightnessThreadPool(50);
            processor.saveImage("output_brightness_threadpool.png");

            // Obliczanie histogramu dla kanału czerwonego (0)
            int[] histogram = processor.calculateHistogram(0);

            // Tworzenie obrazu histogramu
            BufferedImage histogramImage = processor.createHistogramImage(histogram);
            processor.setImage(histogramImage);
            processor.saveImage("histogram.png");

        } catch (IOException | InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}


