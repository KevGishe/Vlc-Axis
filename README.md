# Vlc-Axis
Using vlcj to stream RTSP from Axis cameras
package vlcplay;

import java.awt.BorderLayout;
import java.awt.Canvas;
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.List;

import javax.imageio.ImageIO;
import javax.swing.BoxLayout;
import javax.swing.ImageIcon;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.JTextField;
import javax.swing.SwingUtilities;
import javax.swing.border.EmptyBorder;

import com.sun.jna.NativeLibrary;

import uk.co.caprica.vlcj.binding.internal.libvlc_media_t;
import uk.co.caprica.vlcj.player.MediaPlayer;
import uk.co.caprica.vlcj.player.MediaPlayerEventAdapter;
import uk.co.caprica.vlcj.player.MediaPlayerFactory;
import uk.co.caprica.vlcj.player.embedded.EmbeddedMediaPlayer;
import uk.co.caprica.vlcj.runtime.RuntimeUtil;
import uk.co.caprica.vlcj.test.VlcjTest;

/**
 * A  camview player implemented in JAVA using vlcj, swing and vlc
 * The URL/MRL must be in the following format:
 *   http://192.168.20.252/axis-cgi/mjpg/video.cgi
 * 
 */
public class CamviewPlayer extends VlcjTest {

    private final MediaPlayerFactory factory;
    private final EmbeddedMediaPlayer mediaPlayer;
    private final Frame mainFrame;

    private final JLabel urlLabel;
    private final JTextField urlTextField;
    private final JButton playButton;
    private final JButton pauseButton;
    private final JButton stopButton;
    private final JButton SnapshotButton;

    


    public static void main(String[] args) throws Exception {
       // setLookAndFeel();
        NativeLibrary.addSearchPath(RuntimeUtil.getLibVlcLibraryName(), "C:/Program Files (x86)/VideoLAN/VLC");
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new CamviewPlayer().start();
            }
        });
    }

    public CamviewPlayer() {
        mainFrame = new Frame("Camviewer");
        mainFrame.setIconImage(new ImageIcon(getClass().getResource("/icons/vlcj-logo.png")).getImage());
        mainFrame.setSize(800, 600);
        mainFrame.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                exit(0);
            }
        });
        mainFrame.setLayout(new BorderLayout());

        JPanel cp = new JPanel();
        cp.setBackground(Color.black);
        cp.setLayout(new BorderLayout());

        JPanel ip = new JPanel();
        ip.setBorder(new EmptyBorder(4, 4, 4, 4));
        ip.setLayout(new BoxLayout(ip, BoxLayout.X_AXIS));

        urlLabel = new JLabel("URL:");
        urlLabel.setDisplayedMnemonic('u');
        urlLabel.setToolTipText("Enter a URL in the format http://192.168.20.249/axis-cgi/mjpg/video.cgi");
        urlTextField = new JTextField(40);
        urlTextField.setFocusAccelerator('u');
        urlTextField.setToolTipText("Enter a URL in the format http://192.168.20.249/axis-cgi/mjpg/video.cgi");
        playButton = new JButton("Play");
        playButton.setMnemonic('p');
        pauseButton = new JButton("Pause");
        pauseButton.setMnemonic('q');
        stopButton = new JButton("Stop");
        stopButton.setMnemonic('r');
        SnapshotButton = new JButton("Snapshot");
        SnapshotButton.setMnemonic('s');
        

        ip.add(urlLabel);
        ip.add(urlTextField);
        ip.add(playButton);
        ip.add(pauseButton);
        ip.add(stopButton);
        ip.add(SnapshotButton);
        

        cp.add(ip, BorderLayout.NORTH);

        Canvas vs = new Canvas();
        vs.setBackground(Color.black);
        cp.add(vs, BorderLayout.CENTER);

        mainFrame.add(cp, BorderLayout.CENTER);

        urlTextField.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                play();
            }
        });

        playButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
            	 //if(mediaPlayer.isPlaying()) mediaPlayer.pause(); else mediaPlayer.play();
            	play();
            }
        });
        
        stopButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                mediaPlayer.stop();
            }
        });

        pauseButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                mediaPlayer.pause();
            }
        });
        
        SnapshotButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
            	BufferedImage image1 = mediaPlayer.getSnapshot(600,400);
            	 show("Image getSnapshot", image1, 1);
            	 
            	 File file1 = new File("snapshot.jpg");
            	 try {
					ImageIO.write(image1,"jpg", file1);
				} catch (IOException e1) {
					// TODO Auto-generated catch block
					e1.printStackTrace();
				}
            }
        });
        

        factory = new MediaPlayerFactory();

        mediaPlayer = factory.newEmbeddedMediaPlayer();
        mediaPlayer.setVideoSurface(factory.newVideoSurface(vs));

        mediaPlayer.setPlaySubItems(true); 
        mediaPlayer.addMediaPlayerEventListener(new MediaPlayerEventAdapter() {
            @Override
            public void buffering(MediaPlayer mediaPlayer, float newCache) {
                System.out.println("Buffering " + newCache);
            }

            @Override
            public void mediaSubItemAdded(MediaPlayer mediaPlayer, libvlc_media_t subItem) {
                List<String> items = mediaPlayer.subItems();
                System.out.println(items);
            }
        });
    }

    private void start() {
        mainFrame.setVisible(true);
    }

    private void play() {
        String mrl = urlTextField.getText();
        mediaPlayer.playMedia(mrl);
    }

    private void exit(int value) {
        mediaPlayer.stop();
        mediaPlayer.release();
        factory.release();
        System.exit(value);
    }
    
    private static void show(String title, final BufferedImage img, int i) {
        JFrame f = new JFrame(title);
        f.setDefaultCloseOperation(JFrame.HIDE_ON_CLOSE);
        f.setContentPane(new JPanel() {
            @Override
            protected void paintChildren(Graphics g) {
                Graphics2D g2 = (Graphics2D)g;
                g2.drawImage(img, null, 0, 0);
            }

            @Override
            public Dimension getPreferredSize() {
                return new Dimension(img.getWidth(), img.getHeight());
            }
        });
        f.pack();
        f.setLocation(50 + (i * 50), 50 + (i * 50));
        f.setVisible(true);
    }
}
