# EEG-Based-Attention-Detection-System-for-BCI-Applications
%% ============================================================
% EEG-BASED ATTENTION DETECTION SYSTEM FOR BCI APPLICATIONS
%
% MATLAB implementation
%
% Outputs:
% 1. Raw EEG Signal
% 2. Preprocessed EEG Signal
% 3. Frequency Spectrum / PSD
% 4. EEG Band Power Distribution
% 5. Attention Classification
%
% Classes:
% Focused / Unfocused / Drowsy
%% ============================================================

clc;
clearvars;
close all;

%% ============================================================
% 1. SELECT EEG DATASET
%% ============================================================

[fileName,filePath] = uigetfile('*.mat', ...
    'Select EEG MAT Dataset');

if isequal(fileName,0)
    error('No EEG dataset was selected.');
end

fullFileName = fullfile(filePath,fileName);

fprintf('\n============================================\n');
fprintf('       EEG ATTENTION DETECTION SYSTEM\n');
fprintf('============================================\n');

fprintf('\nLoading file: %s\n',fileName);

% Use a variable name other than "data"
EEGFile = load(fullFileName);

%% ============================================================
% 2. FIND ALL POSSIBLE NUMERIC EEG MATRICES
%% ============================================================

fprintf('\nSearching for EEG signal inside MAT file...\n');

candidates = {};
candidateNames = {};

topFields = fieldnames(EEGFile);

for k = 1:length(topFields)

    currentValue = EEGFile.(topFields{k});

    [tempData,tempNames] = findNumericMatrices( ...
        currentValue,topFields{k});

    for n = 1:length(tempData)

        candidates{end+1} = tempData{n}; %#ok<SAGROW>
        candidateNames{end+1} = tempNames{n}; %#ok<SAGROW>

    end
end


%% ============================================================
% 3. DISPLAY CANDIDATES
%% ============================================================

fprintf('\n--------------------------------------------\n');
fprintf('Possible EEG data found:\n');
fprintf('--------------------------------------------\n');

for k = 1:length(candidates)

    temp = candidates{k};

    fprintf('%d. %-30s Size: ', ...
        k,candidateNames{k});

    fprintf('%dx%d',size(temp,1),size(temp,2));

    fprintf('  Variance: %.6g\n', ...
        var(double(temp(:)),0));

end


%% ============================================================
% 4. SELECT BEST EEG MATRIX
%% ============================================================

if isempty(candidates)

    error(['No numeric matrix was found inside the MAT file. ' ...
           'The dataset format is not supported by this code.']);

end


% Score every candidate
scores = zeros(1,length(candidates));

for k = 1:length(candidates)

    temp = double(candidates{k});

    if isempty(temp)
        scores(k) = -Inf;
        continue;
    end

    numberValues = numel(temp);

    signalVariance = var(temp(:),0);

    signalRange = max(temp(:)) - min(temp(:));

    % Reject completely constant data
    if signalVariance <= eps || signalRange <= eps

        scores(k) = -Inf;
        continue;

    end

    % Prefer large matrices
    sizeScore = log10(numberValues + 1);

    % Prefer matrices with multiple channels
    [r,c] = size(temp);

    if r >= 1000 && c >= 2
        channelScore = 5;
    elseif c >= 2
        channelScore = 2;
    else
        channelScore = 0;
    end

    % Prefer long signals
    if max(r,c) >= 10000
        lengthScore = 5;
    elseif max(r,c) >= 1000
        lengthScore = 3;
    else
        lengthScore = 0;
    end

    scores(k) = ...
        sizeScore + channelScore + lengthScore + ...
        log10(abs(signalVariance) + 1);

end


[bestScore,bestIndex] = max(scores);


if ~isfinite(bestScore)

    error(['All numeric variables in the MAT file are constant ' ...
           'or empty. The actual EEG signal could not be identified.']);

end


EEG = double(candidates{bestIndex});

selectedVariable = candidateNames{bestIndex};

fprintf('\n============================================\n');
fprintf('SELECTED EEG VARIABLE\n');
fprintf('============================================\n');

fprintf('Variable: %s\n',selectedVariable);

fprintf('Original size: %d x %d\n', ...
    size(EEG,1),size(EEG,2));


%% ============================================================
% 5. ARRANGE DATA
% Rows = samples
% Columns = channels
%% ============================================================

[r,c] = size(EEG);

if r < c

    EEG = EEG';

end

totalSamples = size(EEG,1);
totalChannels = size(EEG,2);

fprintf('Final EEG size: %d samples x %d channels\n', ...
    totalSamples,totalChannels);


%% ============================================================
% 6. REMOVE INVALID VALUES
%% ============================================================

EEG(~isfinite(EEG)) = NaN;

for ch = 1:totalChannels

    channelData = EEG(:,ch);

    medValue = median(channelData,'omitnan');

    if isnan(medValue)
        medValue = 0;
    end

    channelData(isnan(channelData)) = medValue;

    EEG(:,ch) = channelData;

end


%% ============================================================
% 7. CHECK SIGNAL QUALITY
%% ============================================================

signalVariance = var(EEG,0,1);

fprintf('\nChannel variances:\n');

disp(signalVariance);


if all(signalVariance < eps)

    error(['The selected matrix contains no EEG variation. ' ...
           'Please check the MAT-file structure.']);

end


%% ============================================================
% 8. SAMPLING FREQUENCY
%% ============================================================

% Your reference EEG dataset uses 128 Hz.

Fs = 128;

fprintf('Sampling Frequency = %d Hz\n',Fs);


%% ============================================================
% 9. SELECT EEG CHANNEL
%% ============================================================

% Prefer channel 10 because your reference screenshot
% uses O1 (Channel 10).

if totalChannels >= 10

    channelNumber = 10;

else

    channelNumber = 1;

end

fprintf('Selected EEG Channel = %d\n',channelNumber);


rawEEG = EEG(:,channelNumber);


%% ============================================================
% 10. NORMALIZE SIGNAL FOR VISUALIZATION
%% ============================================================

% Keep original signal for analysis.
% Create a normalized version for clearer visualization.

rawMean = mean(rawEEG);

rawStd = std(rawEEG);

if rawStd > eps

    displayEEG = ...
        (rawEEG - rawMean) / rawStd;

else

    displayEEG = rawEEG - rawMean;

end


%% ============================================================
% 11. SELECT 10-SECOND WINDOW
%% ============================================================

analysisDuration = 10;

numberOfSamples = min( ...
    length(displayEEG), ...
    round(analysisDuration*Fs));


displayEEG = displayEEG(1:numberOfSamples);

rawForProcessing = displayEEG;

time = (0:numberOfSamples-1)/Fs;

actualDuration = numberOfSamples/Fs;


fprintf('Analysis duration = %.2f seconds\n', ...
    actualDuration);


%% ============================================================
% 12. OUTPUT 1 - RAW EEG
%% ============================================================

figure( ...
    'Name','Raw EEG Signal', ...
    'NumberTitle','off', ...
    'Color','white');

plot(time,rawForProcessing, ...
    'LineWidth',1);

xlabel('Time (seconds)');
ylabel('Normalized Amplitude');

title('Raw EEG Signal');

grid on;

xlim([0 actualDuration]);


%% ============================================================
% 13. PREPROCESSING
%% ============================================================

% Remove DC component

detrendedEEG = detrend(rawForProcessing);


% Band-pass filter
%
% EEG frequency range:
% 0.5 - 50 Hz

lowCutoff = 0.5;

highCutoff = min(50,Fs/2-1);


[b,a] = butter(4, ...
    [lowCutoff highCutoff]/(Fs/2), ...
    'bandpass');


preprocessedEEG = filtfilt( ...
    b,a,detrendedEEG);


%% ============================================================
% 14. OUTPUT 2 - PREPROCESSED EEG
%% ============================================================

figure( ...
    'Name','Preprocessed EEG Signal', ...
    'NumberTitle','off', ...
    'Color','white');

plot(time,preprocessedEEG, ...
    'LineWidth',1);

xlabel('Time (seconds)');
ylabel('Amplitude');

title('Preprocessed EEG Signal');

grid on;

xlim([0 actualDuration]);


%% ============================================================
% 15. POWER SPECTRAL DENSITY
%% ============================================================

windowLength = min(4*Fs,length(preprocessedEEG));

window = hamming(windowLength);

overlap = round(0.5*windowLength);

nfft = 2048;


[powerSpectrum,frequency] = pwelch( ...
    preprocessedEEG, ...
    window, ...
    overlap, ...
    nfft, ...
    Fs);


%% ============================================================
% 16. OUTPUT 3 - FREQUENCY SPECTRUM
%% ============================================================

powerdB = 10*log10(powerSpectrum + eps);


figure( ...
    'Name','Frequency Spectrum', ...
    'NumberTitle','off', ...
    'Color','white');

plot(frequency,powerdB, ...
    'LineWidth',1.2);

xlabel('Frequency (Hz)');
ylabel('Power (dB/Hz)');

title('Frequency Spectrum (Power Spectral Density)');

grid on;

xlim([0 60]);


%% ============================================================
% 17. EEG FREQUENCY BANDS
%% ============================================================

% Delta : 0.5 - 4 Hz
% Theta : 4 - 8 Hz
% Alpha : 8 - 13 Hz
% Beta  : 13 - 30 Hz
% Gamma : 30 - 50 Hz


bandNames = { ...
    'Delta';
    'Theta';
    'Alpha';
    'Beta';
    'Gamma'};


bandLimits = [ ...
    0.5  4;
    4    8;
    8   13;
    13  30;
    30  50];


%% ============================================================
% 18. CALCULATE BAND POWER
%% ============================================================

bandPower = zeros(1,5);


for i = 1:5

    lowFreq = bandLimits(i,1);

    highFreq = bandLimits(i,2);

    index = ...
        frequency >= lowFreq & ...
        frequency < highFreq;


    if any(index)

        bandPower(i) = trapz( ...
            frequency(index), ...
            powerSpectrum(index));

    else

        bandPower(i) = 0;

    end

end


%% ============================================================
% 19. RELATIVE BAND POWER
%% ============================================================

totalBandPower = sum(bandPower);


if totalBandPower > 0

    relativeBandPower = ...
        bandPower ./ totalBandPower;

else

    relativeBandPower = zeros(1,5);

end


%% ============================================================
% 20. OUTPUT 4 - EEG BAND POWER
%% ============================================================

figure( ...
    'Name','EEG Band Power Distribution', ...
    'NumberTitle','off', ...
    'Color','white');


bar(relativeBandPower);


xlabel('Frequency Bands');

ylabel('Relative Power');

title('EEG Band Power Distribution');


set(gca, ...
    'XTick',1:5, ...
    'XTickLabel',bandNames);


grid on;


%% ============================================================
% 21. DISPLAY BAND POWER VALUES
%% ============================================================

fprintf('\n============================================\n');
fprintf('         EEG BAND POWER RESULTS\n');
fprintf('============================================\n');


for i = 1:5

    fprintf('%-8s : %.4f  (%6.2f%%)\n', ...
        bandNames{i}, ...
        relativeBandPower(i), ...
        relativeBandPower(i)*100);

end


%% ============================================================
% 22. EXTRACT ATTENTION FEATURES
%% ============================================================

deltaPower = relativeBandPower(1);

thetaPower = relativeBandPower(2);

alphaPower = relativeBandPower(3);

betaPower = relativeBandPower(4);

gammaPower = relativeBandPower(5);


% Attention-related feature

attentionIndex = ...
    betaPower / ...
    (alphaPower + thetaPower + eps);


%% ============================================================
% 23. ATTENTION CLASSIFICATION
%% ============================================================

% Simple demonstration classifier.
%
% NOTE:
% This is not a clinically validated attention detector.
% A trained SVM model should be used when labeled attention
% data is available.


if attentionIndex >= 0.60

    attentionState = 'Focused';

elseif attentionIndex >= 0.35

    attentionState = 'Unfocused';

else

    attentionState = 'Drowsy';

end


%% ============================================================
% 24. CONFIDENCE ESTIMATION
%% ============================================================

% Simple confidence display for demonstration.

if strcmp(attentionState,'Focused')

    confidence = ...
        min(99,60 + attentionIndex*40);

elseif strcmp(attentionState,'Unfocused')

    confidence = ...
        min(99,60 + abs(attentionIndex-0.35)*80);

else

    confidence = ...
        min(99,60 + abs(0.35-attentionIndex)*80);

end


%% ============================================================
% 25. DISPLAY CLASSIFICATION RESULT
%% ============================================================

fprintf('\n============================================\n');
fprintf('          ATTENTION CLASSIFICATION\n');
fprintf('============================================\n');

fprintf('Attention Index : %.4f\n', ...
    attentionIndex);

fprintf('Predicted State : %s\n', ...
    attentionState);

fprintf('Confidence      : %.1f%%\n', ...
    confidence);


%% ============================================================
% 26. COMBINED RESULT FIGURE
%% ============================================================

figure( ...
    'Name','EEG Attention Detection Summary', ...
    'NumberTitle','off', ...
    'Color','white');


%% RAW EEG

subplot(2,2,1);

plot(time,rawForProcessing, ...
    'LineWidth',1);

xlabel('Time (s)');
ylabel('Amplitude');

title('Raw EEG Signal');

grid on;


%% PREPROCESSED EEG

subplot(2,2,2);

plot(time,preprocessedEEG, ...
    'LineWidth',1);

xlabel('Time (s)');
ylabel('Amplitude');

title('Preprocessed EEG Signal');

grid on;


%% FREQUENCY SPECTRUM

subplot(2,2,3);

plot(frequency,powerdB, ...
    'LineWidth',1);

xlabel('Frequency (Hz)');
ylabel('Power (dB/Hz)');

title('Frequency Spectrum');

xlim([0 60]);

grid on;


%% BAND POWER

subplot(2,2,4);

bar(relativeBandPower);

xlabel('Frequency Bands');

ylabel('Relative Power');

title('Band Power Distribution');

set(gca, ...
    'XTick',1:5, ...
    'XTickLabel',bandNames);

grid on;


%% ============================================================
% 27. FINAL RESULT
%% ============================================================

fprintf('\n============================================\n');
fprintf('             ANALYSIS COMPLETED\n');
fprintf('============================================\n');

fprintf('\nOutputs generated:\n');

fprintf('1. Raw EEG Signal\n');
fprintf('2. Preprocessed EEG Signal\n');
fprintf('3. Frequency Spectrum (PSD)\n');
fprintf('4. EEG Band Power Distribution\n');

fprintf('\n--------------------------------------------\n');

fprintf('Predicted Attention: %s\n', ...
    attentionState);

fprintf('Confidence: %.1f%%\n',confidence);

fprintf('--------------------------------------------\n');


%% ============================================================
% LOCAL FUNCTION
% FIND NUMERIC MATRICES INSIDE STRUCTURES/CELLS
%% ============================================================

function [matrices,names] = ...
    findNumericMatrices(inputValue,currentName)

matrices = {};
names = {};


%% DIRECT NUMERIC DATA

if isnumeric(inputValue)

    if isempty(inputValue)

        return;

    end


    if ismatrix(inputValue)

        [rows,columns] = size(inputValue);


        % Accept long vectors/matrices

        if numel(inputValue) >= 500

            matrices{end+1} = inputValue;

            names{end+1} = currentName;

        end

    end

    return;

end


%% STRUCTURE

if isstruct(inputValue)

    fields = fieldnames(inputValue);


    for s = 1:numel(inputValue)

        for k = 1:length(fields)

            fieldName = fields{k};


            try

                fieldValue = ...
                    inputValue(s).(fieldName);


                if numel(inputValue) > 1

                    newName = sprintf( ...
                        '%s(%d).%s', ...
                        currentName,s,fieldName);

                else

                    newName = sprintf( ...
                        '%s.%s', ...
                        currentName,fieldName);

                end


                [newMatrices,newNames] = ...
                    findNumericMatrices( ...
                    fieldValue,newName);


                if ~isempty(newMatrices)

                    matrices = ...
                        [matrices,newMatrices]; %#ok<AGROW>

                    names = ...
                        [names,newNames]; %#ok<AGROW>

                end

            catch

            end

        end

    end

    return;

end


%% CELL ARRAY

if iscell(inputValue)

    for k = 1:numel(inputValue)

        try

            newName = sprintf( ...
                '%s{%d}',currentName,k);


            [newMatrices,newNames] = ...
                findNumericMatrices( ...
                inputValue{k},newName);


            if ~isempty(newMatrices)

                matrices = ...
                    [matrices,newMatrices]; %#ok<AGROW>

                names = ...
                    [names,newNames]; %#ok<AGROW>

            end

        catch

        end

    end

end

end
