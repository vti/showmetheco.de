Title: Learning music notes with Perl
Tags: Perl, music, notes

So the other day I bought a MIDI2USB interface to my piano keyboard and as
a programmer I was curious about sending back and forth those MIDI events.
I decided to write a simple Perl game that can help to learn the music notes.
The routine is simple. The random note is generated and printed out on the
terminal (like D3), then you have to find that note on the keyboard and play it.
If it matches you get ok, otherwise not ok. And so on. As always
[CPAN](http://metacpan.org) saved hours of my time.

[cut]

For receiving and sending MIDI events I used [MIDI::ALSA](https://metacpan.org/pod/MIDI::ALSA) which is a wrapper
around alsa library (since I'm on GNU/Linux).

    use MIDI::ALSA qw(
      SND_SEQ_EVENT_PORT_UNSUBSCRIBED
      SND_SEQ_EVENT_NOTEON
      SND_SEQ_EVENT_NOTEOFF
    );

    MIDI::ALSA::client('Perl MIDI::ALSA client', 1, 1, 0);

    MIDI::ALSA::connectfrom(0, 20, 0) or die "Can't connect: $!";

    MIDI::ALSA::start() or die "Can't start: $!";

    while (1) {
        my @alsaevent = MIDI::ALSA::input();

        my @data = @{$alsaevent[7]};

        if ($alsaevent[0] == SND_SEQ_EVENT_PORT_UNSUBSCRIBED()) {
            last;
        }
        elsif ($alsaevent[0] == SND_SEQ_EVENT_NOTEOFF()
            || ($alsaevent[0] == SND_SEQ_EVENT_NOTEON && !$data[2]))
        {
            ... just ignore the key being released ...
        }
        elsif ($alsaevent[0] == SND_SEQ_EVENT_NOTEON()) {
            my $channel = $data[0];
            my $pitch   = $data[1];
            my $key     = $channel * 128 + $pitch;

            ... we got the $key being pressed ...
        }
    }

`connectfrom` is a bit confusing, since it is actually the input port. In order
to get the correct input port run `aseqdump -l` and you will get smth like:

    Port    Client name                      Port name
    0:0     System                           Timer
    0:1     System                           Announce
    14:0    Midi Through                     Midi Through Port-0
    20:0    CME U2MIDI                       CME U2MIDI MIDI 1

Where in my case `20:0` is a MIDI to USB interface and `connectform` becomes:

    MIDI::ALSA::connectfrom(0, 20, 0)

Once we can receive MIDI events, we can write our random notes generator. My
keyboard is 76 keys, so I need values from 28 to 103. You can easily get you
own key numbers just by playing the left-most and the right-most keys and
seeing the result. Maybe you can get those values automatically via MIDI
events, I haven't tried it myself.

So the random note is:

    my $min = 28;
    my $max = 103;

    int(rand($max - $min + 1) + $min);

For displaying a note I used [Music::Note](https://metacpan.org/pod/Music::Note). A wonderful module that can
transform a MIDI number into a human readable format.

    my $note = Music::Note->new($random_note, 'midinum');

    my $string = '';
    $string .= $note->step;
    if ($note->alter) {
        $string .= '#';
    }
    $string .= $note->octave;

    # or as Ben Daglish (author of Music::Note)
    # suggested in comments below:
    $note->format('iso')

So in case of 67 we get G4, and of 68 we get G#4, and so on.

Have fun with Perl and MIDI!
